# Multi-Tenant Authentication Patterns 2025

## Multi-Tenancy Overview

Multi-tenancy is an architecture where a single instance of software serves multiple customers (tenants), with each tenant's data and configuration isolated from others. In 2025, multi-tenant authentication has evolved to support complex B2B SaaS scenarios with sophisticated organization hierarchies and fine-grained access controls.

## Multi-Tenant Architecture Patterns

### 1. Database-Level Multi-Tenancy

#### Schema-Per-Tenant
```sql
-- Separate schema for each tenant
CREATE SCHEMA tenant_acme;
CREATE SCHEMA tenant_widgets_inc;

-- Tables in each schema
CREATE TABLE tenant_acme.users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE tenant_widgets_inc.users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### Shared Database with Tenant ID
```sql
-- Shared tables with tenant isolation
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  slug TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  settings JSONB DEFAULT '{}',
  subscription_tier TEXT DEFAULT 'free',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
  email TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(tenant_id, email)
);

-- Row Level Security for tenant isolation
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can only see their tenant's data" ON users
  FOR ALL USING (
    tenant_id = current_setting('app.current_tenant_id')::UUID
  );

-- Set tenant context function
CREATE OR REPLACE FUNCTION set_tenant_context(tenant_uuid UUID)
RETURNS void AS $$
BEGIN
  PERFORM set_config('app.current_tenant_id', tenant_uuid::text, true);
END;
$$ LANGUAGE plpgsql;
```

### 2. Application-Level Multi-Tenancy

#### Tenant Resolution Middleware
```typescript
// Tenant resolution strategies
export enum TenantResolutionStrategy {
  SUBDOMAIN = 'subdomain',
  PATH_PREFIX = 'path_prefix',
  HEADER = 'header',
  JWT_CLAIM = 'jwt_claim'
}

export interface Tenant {
  id: string
  slug: string
  name: string
  domain?: string
  settings: {
    authentication: {
      ssoEnabled: boolean
      mfaRequired: boolean
      passwordPolicy?: PasswordPolicy
    }
    features: string[]
    branding?: {
      logo?: string
      primaryColor?: string
      customDomain?: string
    }
  }
}

export class TenantResolver {
  constructor(
    private strategy: TenantResolutionStrategy,
    private tenantService: TenantService
  ) {}

  async resolveTenant(req: Request): Promise<Tenant | null> {
    let tenantIdentifier: string | null = null

    switch (this.strategy) {
      case TenantResolutionStrategy.SUBDOMAIN:
        tenantIdentifier = this.extractSubdomain(req.get('host') || '')
        break
        
      case TenantResolutionStrategy.PATH_PREFIX:
        tenantIdentifier = this.extractPathPrefix(req.path)
        break
        
      case TenantResolutionStrategy.HEADER:
        tenantIdentifier = req.get('x-tenant-id') || null
        break
        
      case TenantResolutionStrategy.JWT_CLAIM:
        tenantIdentifier = this.extractTenantFromJWT(req.get('authorization'))
        break
    }

    if (!tenantIdentifier) {
      return null
    }

    return await this.tenantService.findBySlug(tenantIdentifier)
  }

  private extractSubdomain(host: string): string | null {
    // Extract tenant from subdomain: tenant.app.com -> tenant
    const parts = host.split('.')
    if (parts.length >= 3 && parts[0] !== 'www') {
      return parts[0]
    }
    return null
  }

  private extractPathPrefix(path: string): string | null {
    // Extract tenant from path: /tenant/dashboard -> tenant
    const match = path.match(/^\/([^\/]+)/)
    return match ? match[1] : null
  }

  private extractTenantFromJWT(authHeader?: string): string | null {
    if (!authHeader?.startsWith('Bearer ')) return null
    
    try {
      const token = authHeader.substring(7)
      const decoded = jwt.decode(token) as any
      return decoded?.tenant_id || null
    } catch {
      return null
    }
  }
}

// Express middleware for tenant resolution
export function tenantMiddleware(
  resolver: TenantResolver
) {
  return async (req: Request & { tenant?: Tenant }, res: Response, next: NextFunction) => {
    try {
      const tenant = await resolver.resolveTenant(req)
      
      if (!tenant) {
        return res.status(404).json({ error: 'Tenant not found' })
      }

      req.tenant = tenant
      
      // Set database context for RLS
      if (req.db) {
        await req.db.query('SELECT set_tenant_context($1)', [tenant.id])
      }

      next()
    } catch (error) {
      res.status(500).json({ error: 'Tenant resolution failed' })
    }
  }
}
```

### 3. Hierarchical Organizations

#### Nested Organization Structure
```typescript
export interface Organization {
  id: string
  name: string
  slug: string
  parentId?: string // For nested organizations
  type: 'root' | 'department' | 'team' | 'project'
  metadata: {
    industry?: string
    size?: 'startup' | 'small' | 'medium' | 'enterprise'
    location?: string
  }
  settings: OrganizationSettings
  created_at: Date
  updated_at: Date
}

export interface OrganizationMembership {
  id: string
  userId: string
  organizationId: string
  role: string
  permissions: string[]
  inheritFromParent: boolean
  joinedAt: Date
  invitedBy?: string
}

export class HierarchicalOrganizationService {
  async createOrganization(data: {
    name: string
    slug: string
    parentId?: string
    type: Organization['type']
    createdBy: string
  }): Promise<Organization> {
    const organization = await this.db.organization.create({
      data: {
        ...data,
        settings: this.getDefaultSettings(data.type)
      }
    })

    // Auto-assign creator as admin
    await this.addMember(organization.id, data.createdBy, 'admin')

    // Inherit certain settings from parent
    if (data.parentId) {
      await this.inheritFromParent(organization.id, data.parentId)
    }

    return organization
  }

  async getUserOrganizations(userId: string): Promise<Organization[]> {
    // Get all organizations user belongs to, including inherited access
    const directMemberships = await this.db.organizationMembership.findMany({
      where: { userId },
      include: { organization: true }
    })

    const inheritedOrganizations = await this.getInheritedOrganizations(userId)
    
    return [...directMemberships.map(m => m.organization), ...inheritedOrganizations]
  }

  async getOrganizationHierarchy(organizationId: string): Promise<Organization[]> {
    const hierarchy: Organization[] = []
    let currentId: string | null = organizationId

    // Walk up the hierarchy
    while (currentId) {
      const org = await this.db.organization.findUnique({
        where: { id: currentId }
      })

      if (!org) break

      hierarchy.unshift(org) // Add to beginning
      currentId = org.parentId
    }

    return hierarchy
  }

  async getChildOrganizations(
    organizationId: string,
    includeDescendants = false
  ): Promise<Organization[]> {
    if (includeDescendants) {
      // Recursive query for all descendants
      return await this.db.$queryRaw`
        WITH RECURSIVE org_tree AS (
          SELECT * FROM organizations WHERE parent_id = ${organizationId}
          UNION ALL
          SELECT o.* FROM organizations o
          INNER JOIN org_tree ot ON o.parent_id = ot.id
        )
        SELECT * FROM org_tree
      `
    } else {
      // Direct children only
      return await this.db.organization.findMany({
        where: { parentId: organizationId }
      })
    }
  }

  async hasPermissionInHierarchy(
    userId: string,
    organizationId: string,
    permission: string
  ): Promise<boolean> {
    const hierarchy = await this.getOrganizationHierarchy(organizationId)
    
    // Check permissions from root to current organization
    for (const org of hierarchy) {
      const membership = await this.db.organizationMembership.findUnique({
        where: {
          userId_organizationId: {
            userId,
            organizationId: org.id
          }
        }
      })

      if (membership?.permissions.includes(permission) || 
          membership?.permissions.includes('*')) {
        return true
      }
    }

    return false
  }

  private async getInheritedOrganizations(userId: string): Promise<Organization[]> {
    // Complex query to find organizations user has access to through hierarchy
    return []
  }

  private async inheritFromParent(organizationId: string, parentId: string) {
    const parent = await this.db.organization.findUnique({
      where: { id: parentId }
    })

    if (parent) {
      // Inherit specific settings
      await this.db.organization.update({
        where: { id: organizationId },
        data: {
          settings: {
            ...parent.settings,
            // Override specific settings for child
          }
        }
      })
    }
  }
}
```

## Multi-Tenant Authentication Flows

### 1. Clerk Multi-Tenant Implementation

```typescript
// Advanced Clerk multi-tenant setup
import { ClerkProvider, useOrganization, useOrganizationList, useUser } from '@clerk/nextjs'
import { useRouter } from 'next/navigation'
import { useEffect, useState } from 'react'

// Tenant-aware Clerk provider
export function MultiTenantClerkProvider({ 
  children,
  tenantSlug 
}: { 
  children: React.ReactNode
  tenantSlug?: string
}) {
  return (
    <ClerkProvider
      appearance={{
        variables: {
          colorPrimary: '#2563eb' // Can be customized per tenant
        },
        elements: {
          card: "shadow-lg border border-gray-200",
          headerTitle: "text-2xl font-bold",
          socialButtonsBlockButton: "border border-gray-300 hover:bg-gray-50"
        }
      }}
      // Tenant-specific sign-in URL
      signInUrl={tenantSlug ? `/${tenantSlug}/sign-in` : '/sign-in'}
      signUpUrl={tenantSlug ? `/${tenantSlug}/sign-up` : '/sign-up'}
      afterSignInUrl={tenantSlug ? `/${tenantSlug}/dashboard` : '/dashboard'}
      afterSignUpUrl={tenantSlug ? `/${tenantSlug}/dashboard` : '/dashboard'}
    >
      <TenantContextProvider tenantSlug={tenantSlug}>
        {children}
      </TenantContextProvider>
    </ClerkProvider>
  )
}

// Tenant context for additional tenant-specific data
interface TenantContext {
  tenant: Tenant | null
  isLoading: boolean
  switchTenant: (tenantSlug: string) => Promise<void>
}

const TenantContext = createContext<TenantContext | undefined>(undefined)

export function TenantContextProvider({ 
  children, 
  tenantSlug 
}: { 
  children: React.ReactNode
  tenantSlug?: string
}) {
  const [tenant, setTenant] = useState<Tenant | null>(null)
  const [isLoading, setIsLoading] = useState(true)
  const router = useRouter()
  const { user } = useUser()

  useEffect(() => {
    if (tenantSlug && user) {
      fetchTenant(tenantSlug)
    } else {
      setIsLoading(false)
    }
  }, [tenantSlug, user])

  const fetchTenant = async (slug: string) => {
    setIsLoading(true)
    try {
      const response = await fetch(`/api/tenants/${slug}`)
      if (response.ok) {
        const tenantData = await response.json()
        setTenant(tenantData)
      }
    } catch (error) {
      console.error('Failed to fetch tenant:', error)
    } finally {
      setIsLoading(false)
    }
  }

  const switchTenant = async (newTenantSlug: string) => {
    router.push(`/${newTenantSlug}/dashboard`)
  }

  return (
    <TenantContext.Provider value={{ tenant, isLoading, switchTenant }}>
      {children}
    </TenantContext.Provider>
  )
}

export function useTenant() {
  const context = useContext(TenantContext)
  if (!context) {
    throw new Error('useTenant must be used within TenantContextProvider')
  }
  return context
}

// Multi-tenant organization switcher
export function TenantAwareOrganizationSwitcher() {
  const { organizationList, setActive } = useOrganizationList()
  const { organization } = useOrganization()
  const { tenant } = useTenant()
  const router = useRouter()

  const filteredOrganizations = organizationList?.filter(org => 
    // Only show organizations that belong to current tenant
    org.organization.publicMetadata?.tenantId === tenant?.id
  )

  const handleOrganizationSelect = async (orgId: string) => {
    const selectedOrg = filteredOrganizations?.find(
      org => org.organization.id === orgId
    )

    if (selectedOrg) {
      await setActive({ organization: selectedOrg.organization })
      // Redirect to organization-specific dashboard
      router.push(`/${tenant?.slug}/org/${selectedOrg.organization.slug}/dashboard`)
    }
  }

  return (
    <div className="relative">
      <select
        value={organization?.id || ''}
        onChange={(e) => handleOrganizationSelect(e.target.value)}
        className="block w-full px-3 py-2 border border-gray-300 rounded-md"
      >
        <option value="">Select Organization</option>
        {filteredOrganizations?.map(({ organization: org }) => (
          <option key={org.id} value={org.id}>
            {org.name} ({org.membersCount} members)
          </option>
        ))}
      </select>
    </div>
  )
}

// Tenant-specific authentication pages
export default function TenantSignInPage({ 
  params 
}: { 
  params: { tenant: string } 
}) {
  const { tenant, isLoading } = useTenant()

  if (isLoading) {
    return <div>Loading tenant...</div>
  }

  if (!tenant) {
    return <div>Tenant not found</div>
  }

  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="max-w-md w-full">
        {tenant.settings.branding?.logo && (
          <img 
            src={tenant.settings.branding.logo} 
            alt={tenant.name}
            className="h-12 mx-auto mb-8"
          />
        )}
        
        <SignIn 
          appearance={{
            variables: {
              colorPrimary: tenant.settings.branding?.primaryColor || '#2563eb'
            }
          }}
          redirectUrl={`/${params.tenant}/dashboard`}
        />
        
        <p className="text-center mt-4 text-sm text-gray-600">
          Signing in to {tenant.name}
        </p>
      </div>
    </div>
  )
}
```

### 2. Supabase Multi-Tenant Implementation

```typescript
// Supabase multi-tenant auth with RLS
import { createClientComponentClient } from '@supabase/auth-helpers-nextjs'
import { useRouter } from 'next/navigation'
import { useState, useEffect } from 'react'

export function useMultiTenantAuth(tenantSlug: string) {
  const [tenant, setTenant] = useState<Tenant | null>(null)
  const [user, setUser] = useState<any>(null)
  const [loading, setLoading] = useState(true)
  const supabase = createClientComponentClient()
  const router = useRouter()

  useEffect(() => {
    // Get initial session
    supabase.auth.getSession().then(({ data: { session } }) => {
      setUser(session?.user ?? null)
      if (session?.user) {
        fetchTenantAndSetContext(tenantSlug, session.user.id)
      } else {
        setLoading(false)
      }
    })

    // Listen for auth changes
    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      async (event, session) => {
        setUser(session?.user ?? null)
        
        if (session?.user) {
          await fetchTenantAndSetContext(tenantSlug, session.user.id)
        } else {
          setTenant(null)
          setLoading(false)
        }
      }
    )

    return () => subscription.unsubscribe()
  }, [tenantSlug])

  const fetchTenantAndSetContext = async (slug: string, userId: string) => {
    try {
      // Fetch tenant information
      const { data: tenantData, error: tenantError } = await supabase
        .from('tenants')
        .select('*')
        .eq('slug', slug)
        .single()

      if (tenantError) throw tenantError

      // Verify user has access to this tenant
      const { data: membership, error: membershipError } = await supabase
        .from('tenant_memberships')
        .select('*')
        .eq('tenant_id', tenantData.id)
        .eq('user_id', userId)
        .single()

      if (membershipError) {
        router.push('/unauthorized')
        return
      }

      setTenant(tenantData)
      
      // Set tenant context for RLS
      await supabase.rpc('set_tenant_context', { 
        tenant_uuid: tenantData.id 
      })
      
    } catch (error) {
      console.error('Failed to set tenant context:', error)
      router.push('/tenant-not-found')
    } finally {
      setLoading(false)
    }
  }

  const signInWithTenant = async (email: string, password: string) => {
    const { data, error } = await supabase.auth.signInWithPassword({
      email,
      password
    })

    if (error) throw error

    // After successful sign-in, the auth state change listener will handle tenant setup
    return data
  }

  const signUpWithTenant = async (
    email: string, 
    password: string, 
    metadata?: Record<string, any>
  ) => {
    const { data, error } = await supabase.auth.signUp({
      email,
      password,
      options: {
        data: {
          tenant_slug: tenantSlug,
          ...metadata
        }
      }
    })

    if (error) throw error

    // Create tenant membership after sign-up
    if (data.user) {
      await supabase
        .from('tenant_memberships')
        .insert({
          user_id: data.user.id,
          tenant_id: tenant?.id,
          role: 'member',
          invited_by: null
        })
    }

    return data
  }

  const signOut = async () => {
    await supabase.auth.signOut()
    router.push(`/${tenantSlug}/sign-in`)
  }

  const switchTenant = async (newTenantSlug: string) => {
    if (user) {
      await fetchTenantAndSetContext(newTenantSlug, user.id)
      router.push(`/${newTenantSlug}/dashboard`)
    }
  }

  return {
    tenant,
    user,
    loading,
    signInWithTenant,
    signUpWithTenant,
    signOut,
    switchTenant
  }
}

// Multi-tenant sign-in component
export function MultiTenantSignIn({ tenantSlug }: { tenantSlug: string }) {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [isLoading, setIsLoading] = useState(false)
  const [error, setError] = useState('')
  
  const { tenant, signInWithTenant } = useMultiTenantAuth(tenantSlug)

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setIsLoading(true)
    setError('')

    try {
      await signInWithTenant(email, password)
    } catch (err: any) {
      setError(err.message)
    } finally {
      setIsLoading(false)
    }
  }

  if (!tenant) {
    return <div>Loading tenant...</div>
  }

  return (
    <div className="max-w-md mx-auto">
      <div className="text-center mb-8">
        {tenant.settings.branding?.logo && (
          <img 
            src={tenant.settings.branding.logo} 
            alt={tenant.name}
            className="h-12 mx-auto mb-4"
          />
        )}
        <h1 className="text-2xl font-bold">Sign in to {tenant.name}</h1>
      </div>

      <form onSubmit={handleSubmit} className="space-y-4">
        {error && (
          <div className="bg-red-50 border border-red-200 text-red-700 px-4 py-3 rounded">
            {error}
          </div>
        )}

        <input
          type="email"
          placeholder="Email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          className="w-full px-3 py-2 border border-gray-300 rounded-md"
          required
        />

        <input
          type="password"
          placeholder="Password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          className="w-full px-3 py-2 border border-gray-300 rounded-md"
          required
        />

        <button
          type="submit"
          disabled={isLoading}
          className="w-full bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700 disabled:opacity-50"
          style={{ 
            backgroundColor: tenant.settings.branding?.primaryColor || '#2563eb' 
          }}
        >
          {isLoading ? 'Signing in...' : 'Sign in'}
        </button>
      </form>

      {tenant.settings.authentication.ssoEnabled && (
        <div className="mt-6 pt-6 border-t border-gray-200">
          <button
            onClick={() => {/* Handle SSO */}}
            className="w-full bg-gray-100 text-gray-700 py-2 px-4 rounded-md hover:bg-gray-200"
          >
            Sign in with SSO
          </button>
        </div>
      )}
    </div>
  )
}

// Database functions for tenant context
-- PostgreSQL functions for multi-tenant support
CREATE OR REPLACE FUNCTION set_tenant_context(tenant_uuid UUID)
RETURNS void AS $$
BEGIN
  PERFORM set_config('app.current_tenant_id', tenant_uuid::text, true);
  PERFORM set_config('app.tenant_context_set', 'true', true);
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION get_current_tenant_id()
RETURNS UUID AS $$
BEGIN
  RETURN current_setting('app.current_tenant_id', true)::UUID;
EXCEPTION
  WHEN OTHERS THEN
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Ensure tenant context is set for all operations
CREATE OR REPLACE FUNCTION enforce_tenant_context()
RETURNS TRIGGER AS $$
BEGIN
  IF current_setting('app.tenant_context_set', true) != 'true' THEN
    RAISE EXCEPTION 'Tenant context not set. Call set_tenant_context() first.';
  END IF;
  
  RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

-- Apply trigger to important tables
CREATE TRIGGER enforce_tenant_context_trigger
  BEFORE INSERT OR UPDATE OR DELETE
  ON organizations
  FOR EACH ROW
  EXECUTE FUNCTION enforce_tenant_context();
```

## Multi-Tenant Security Considerations

### 1. Data Isolation
- **Database Level**: Use Row Level Security (RLS) policies
- **Application Level**: Always filter by tenant ID
- **API Level**: Validate tenant access in middleware

### 2. Tenant Context Management
- **Request Scoped**: Set tenant context per request
- **Validation**: Verify user belongs to tenant
- **Auditing**: Log cross-tenant access attempts

### 3. Performance Optimization
- **Connection Pooling**: Tenant-aware connection pools
- **Caching**: Tenant-scoped cache keys
- **Indexing**: Multi-column indexes with tenant ID

### 4. Compliance and Privacy
- **Data Residency**: Tenant-specific data location
- **Backup Strategy**: Per-tenant backup and restore
- **GDPR/CCPA**: Tenant-scoped data deletion

This comprehensive multi-tenant authentication system provides the foundation for building secure, scalable B2B SaaS applications with proper tenant isolation and management.