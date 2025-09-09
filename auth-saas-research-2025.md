# Auth SaaS Research 2025

## Executive Summary

This document provides comprehensive research on authentication-as-a-service (Auth SaaS) solutions for 2025, covering OAuth 2.0/OIDC vs SAML comparison, deep dives into Clerk and Supabase Auth, code examples, RBAC implementations, and organization/multi-tenant patterns.

## Auth SaaS Landscape 2025

### Market Leaders
- **Okta**: Remains the gold standard in enterprise identity with support for nearly every protocol (SAML, OIDC, SCIM, WS-Fed)
- **Microsoft Entra ID**: Delivers smooth authentication with SAML, OIDC and OAuth support
- **WorkOS**: Developer-friendly option designed from the ground up for SaaS apps with native support for SAML, OIDC, SCIM
- **Clerk**: Modern authentication focused on React/Next.js ecosystem
- **Supabase Auth**: Open-source alternative with PostgreSQL-native Row Level Security

### Key Trends for 2025
- **Protocol Evolution**: While OIDC and SAML remain crucial, modern authentication requires more than just protocol support
- **Multi-tenant Focus**: B2B SaaS applications increasingly require organization-based authentication
- **Zero-Trust Security**: Moving beyond basic authentication to comprehensive identity verification
- **Developer Experience**: Emphasis on drop-in components and simplified integration
- **Open Source Growth**: Solutions like Keycloak, Logto, and SuperTokens gaining traction

## OAuth 2.0/OIDC vs SAML Comparison

### Technical Differences

| Aspect | OAuth 2.0/OIDC | SAML |
|--------|----------------|------|
| **Primary Purpose** | Authorization (OAuth), Authentication (OIDC) | Authentication & SSO |
| **Data Format** | JSON Web Tokens (JWT) | XML |
| **Communication** | REST APIs, JSON | SOAP, XML |
| **Mobile Support** | Native mobile support | Limited mobile support |
| **Implementation Complexity** | Simpler, faster implementation | More complex, XML-heavy |
| **Security Features** | Modern, token-based | Mature, XML signatures |

### When to Choose Each (2025)

**Choose OIDC When:**
- Building modern web and mobile applications
- API-centered architecture
- Need fast, easy implementation
- Flexible tech stacks with various end-user devices

**Choose SAML When:**
- Large enterprise environments
- Legacy system integration
- Virtual Desktop Infrastructure (VDI)
- Higher security requirements with multi-factor authentication

**Hybrid Approach:**
Most 2025 implementations benefit from supporting both protocols, with OIDC for modern applications and SAML for enterprise integration.

## Clerk Deep Dive

### Overview
Clerk is a comprehensive authentication and user management platform designed specifically for React, Next.js, and modern web applications.

### Key Features (2025)
- **Multi-tenant Organizations**: Hierarchical relationship between organizations, members, and roles
- **Enterprise SSO**: SAML and OIDC support for identity providers like Okta, Entra ID, Google Workspace
- **RBAC**: Role-based access control with custom roles and permissions
- **Verified Domains**: Restrict membership to specific email domains
- **Drop-in Components**: Pre-built UI components for authentication flows

### Pricing Structure
- **Free Tier**: 10,000 Monthly Active Users (MAUs) and 100 Monthly Active Organizations
- **Pro Plan**: $25/month base cost
- **Add-ons**:
  - Enhanced Authentication: $100/month (MFA, SAML)
  - Enhanced Administration: $100/month (user impersonation)
  - Enhanced B2B SaaS: $100/month (domain restrictions, custom RBAC)

### Code Examples

#### Basic Clerk Setup (Next.js 14)
```typescript
// app/layout.tsx
import { ClerkProvider } from '@clerk/nextjs'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <ClerkProvider>
      <html lang="en">
        <body>{children}</body>
      </html>
    </ClerkProvider>
  )
}
```

#### Authentication Components
```typescript
// app/sign-in/[[...sign-in]]/page.tsx
import { SignIn } from '@clerk/nextjs'

export default function SignInPage() {
  return (
    <div className="flex justify-center py-24">
      <SignIn 
        appearance={{
          elements: {
            formButtonPrimary: "bg-blue-600 hover:bg-blue-500",
            card: "shadow-lg"
          }
        }}
        routing="path"
        path="/sign-in"
        signUpUrl="/sign-up"
      />
    </div>
  )
}
```

#### Organization Management
```typescript
// components/OrganizationSwitcher.tsx
import { OrganizationSwitcher, useOrganization, useUser } from '@clerk/nextjs'

export function OrganizationManager() {
  const { organization, isLoaded: isOrgLoaded } = useOrganization()
  const { user, isLoaded: isUserLoaded } = useUser()

  if (!isOrgLoaded || !isUserLoaded) return <div>Loading...</div>

  return (
    <div className="space-y-4">
      <OrganizationSwitcher 
        appearance={{
          elements: {
            organizationSwitcherTrigger: "border border-gray-300 rounded-lg px-4 py-2"
          }
        }}
        createOrganizationMode="modal"
        organizationProfileMode="modal"
      />
      
      {organization && (
        <div className="bg-gray-50 p-4 rounded-lg">
          <h3 className="font-semibold">Current Organization</h3>
          <p>Name: {organization.name}</p>
          <p>Role: {organization.publicUserData.role}</p>
          <p>Members: {organization.membersCount}</p>
        </div>
      )}
    </div>
  )
}
```

#### RBAC Implementation with Clerk
```typescript
// lib/auth.ts
import { auth, currentUser } from '@clerk/nextjs'
import { redirect } from 'next/navigation'

export async function requireRole(role: string) {
  const { orgRole } = auth()
  
  if (!orgRole || orgRole !== role) {
    redirect('/unauthorized')
  }
}

export async function hasPermission(permission: string): Promise<boolean> {
  const user = await currentUser()
  
  if (!user?.organizationMemberships?.[0]?.permissions) {
    return false
  }
  
  return user.organizationMemberships[0].permissions.includes(permission)
}

// Usage in Server Component
export default async function AdminPage() {
  await requireRole('admin')
  
  return (
    <div>
      <h1>Admin Dashboard</h1>
      {/* Admin content */}
    </div>
  )
}
```

#### Custom Organization Roles
```typescript
// lib/clerk-roles.ts
export const ORGANIZATION_ROLES = {
  ADMIN: {
    key: 'admin',
    name: 'Administrator',
    description: 'Full access to organization settings and members',
    permissions: [
      'org:read',
      'org:write',
      'org:delete',
      'members:read',
      'members:write',
      'members:delete',
      'billing:read',
      'billing:write'
    ]
  },
  MANAGER: {
    key: 'manager', 
    name: 'Manager',
    description: 'Can manage team members and projects',
    permissions: [
      'org:read',
      'members:read',
      'members:write',
      'projects:read',
      'projects:write'
    ]
  },
  MEMBER: {
    key: 'member',
    name: 'Member', 
    description: 'Basic member access',
    permissions: [
      'org:read',
      'projects:read'
    ]
  }
} as const

// Permission checking hook
import { useOrganization } from '@clerk/nextjs'

export function usePermissions() {
  const { organization, membership } = useOrganization()
  
  const hasPermission = (permission: string): boolean => {
    if (!membership?.role) return false
    
    const rolePermissions = Object.values(ORGANIZATION_ROLES)
      .find(role => role.key === membership.role)?.permissions || []
    
    return rolePermissions.includes(permission)
  }
  
  const hasAnyPermission = (permissions: string[]): boolean => {
    return permissions.some(hasPermission)
  }
  
  return { hasPermission, hasAnyPermission }
}
```

## Supabase Auth Deep Dive

### Overview
Supabase Auth is an open-source authentication service built on PostgreSQL with integrated Row Level Security (RLS) for fine-grained access control.

### Key Features (2025)
- **Row Level Security (RLS)**: Database-level authorization
- **Custom Claims & RBAC**: Flexible role-based access control
- **Social & Enterprise Auth**: OAuth providers and SAML SSO
- **Real-time Subscriptions**: Auth-aware real-time data
- **Generous Free Tier**: 10,000 MAUs included

### Pricing Structure
- **Free Tier**: 10,000 MAUs
- **MAU Pricing**: $0.00325 per MAU ($3.25 per 1,000 MAUs)
- **Third-party MAUs**: $0.00325 per Third-Party MAU
- **SSO MAUs**: Additional charges for enterprise SSO

### Code Examples

#### Basic Supabase Auth Setup
```typescript
// lib/supabase.ts
import { createClientComponentClient, createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { Database } from './database.types'

// Client component
export const createClient = () => createClientComponentClient<Database>()

// Server component
export const createServerClient = () => createServerComponentClient<Database>({
  cookies
})
```

#### Authentication Components
```typescript
// components/AuthForm.tsx
'use client'

import { useState } from 'react'
import { createClient } from '@/lib/supabase'
import { useRouter } from 'next/navigation'

export function AuthForm() {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [isLoading, setIsLoading] = useState(false)
  const supabase = createClient()
  const router = useRouter()

  const handleSignIn = async (e: React.FormEvent) => {
    e.preventDefault()
    setIsLoading(true)

    const { data, error } = await supabase.auth.signInWithPassword({
      email,
      password
    })

    if (error) {
      console.error('Auth error:', error.message)
    } else {
      router.push('/dashboard')
      router.refresh()
    }

    setIsLoading(false)
  }

  const handleGoogleSignIn = async () => {
    const { error } = await supabase.auth.signInWithOAuth({
      provider: 'google',
      options: {
        redirectTo: `${window.location.origin}/auth/callback`
      }
    })

    if (error) {
      console.error('OAuth error:', error.message)
    }
  }

  return (
    <div className="max-w-md mx-auto">
      <form onSubmit={handleSignIn} className="space-y-4">
        <input
          type="email"
          placeholder="Email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          className="w-full p-3 border rounded-lg"
          required
        />
        <input
          type="password"
          placeholder="Password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          className="w-full p-3 border rounded-lg"
          required
        />
        <button
          type="submit"
          disabled={isLoading}
          className="w-full bg-blue-600 text-white p-3 rounded-lg hover:bg-blue-700 disabled:opacity-50"
        >
          {isLoading ? 'Signing In...' : 'Sign In'}
        </button>
      </form>

      <div className="mt-4">
        <button
          onClick={handleGoogleSignIn}
          className="w-full bg-white border border-gray-300 p-3 rounded-lg hover:bg-gray-50 flex items-center justify-center"
        >
          <svg className="w-5 h-5 mr-2" viewBox="0 0 24 24">
            {/* Google icon SVG */}
          </svg>
          Continue with Google
        </button>
      </div>
    </div>
  )
}
```

#### RBAC with Custom Claims
```sql
-- Database schema for RBAC
CREATE TABLE user_roles (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  role TEXT NOT NULL CHECK (role IN ('admin', 'manager', 'member')),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(user_id, organization_id)
);

CREATE TABLE organizations (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE organization_members (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  role TEXT NOT NULL DEFAULT 'member',
  joined_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(organization_id, user_id)
);

-- Row Level Security Policies
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE organization_members ENABLE ROW LEVEL SECURITY;

-- Users can only see organizations they're members of
CREATE POLICY "Users can view their organizations" ON organizations
  FOR SELECT USING (
    id IN (
      SELECT organization_id FROM organization_members
      WHERE user_id = auth.uid()
    )
  );

-- Only admins can update organization details
CREATE POLICY "Admins can update organizations" ON organizations
  FOR UPDATE USING (
    id IN (
      SELECT organization_id FROM organization_members
      WHERE user_id = auth.uid() AND role = 'admin'
    )
  );
```

```typescript
// lib/rbac.ts
import { createServerClient } from '@/lib/supabase'

export async function getUserRole(organizationId: string): Promise<string | null> {
  const supabase = createServerClient()
  
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return null

  const { data, error } = await supabase
    .from('organization_members')
    .select('role')
    .eq('user_id', user.id)
    .eq('organization_id', organizationId)
    .single()

  if (error) return null
  return data?.role || null
}

export async function hasPermission(
  organizationId: string,
  requiredRole: 'admin' | 'manager' | 'member'
): Promise<boolean> {
  const userRole = await getUserRole(organizationId)
  if (!userRole) return false

  const roleHierarchy = { admin: 3, manager: 2, member: 1 }
  return roleHierarchy[userRole as keyof typeof roleHierarchy] >= 
         roleHierarchy[requiredRole]
}

// Auth Hook for Custom Claims
export async function createAuthHook(userId: string) {
  const supabase = createServerClient()
  
  // Get user's roles across all organizations
  const { data: roles } = await supabase
    .from('organization_members')
    .select(`
      role,
      organization_id,
      organizations (name, slug)
    `)
    .eq('user_id', userId)

  return {
    user_metadata: {},
    app_metadata: {
      organizations: roles?.map(r => ({
        id: r.organization_id,
        role: r.role,
        name: r.organizations?.name,
        slug: r.organizations?.slug
      })) || []
    }
  }
}
```

#### Row Level Security Examples
```sql
-- Advanced RLS Policies for Multi-tenant SaaS

-- Projects table with organization-based access
CREATE TABLE projects (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  created_by UUID REFERENCES auth.users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

-- Members can view projects in their organizations
CREATE POLICY "Members can view organization projects" ON projects
  FOR SELECT USING (
    organization_id IN (
      SELECT organization_id FROM organization_members
      WHERE user_id = auth.uid()
    )
  );

-- Managers and admins can create projects
CREATE POLICY "Managers can create projects" ON projects
  FOR INSERT WITH CHECK (
    organization_id IN (
      SELECT organization_id FROM organization_members
      WHERE user_id = auth.uid() 
      AND role IN ('admin', 'manager')
    )
  );

-- Only project creators and admins can update projects
CREATE POLICY "Project creators and admins can update" ON projects
  FOR UPDATE USING (
    created_by = auth.uid() OR
    organization_id IN (
      SELECT organization_id FROM organization_members
      WHERE user_id = auth.uid() AND role = 'admin'
    )
  );
```

#### Real-time Auth-aware Subscriptions
```typescript
// hooks/useRealtimeProjects.ts
'use client'

import { useEffect, useState } from 'react'
import { createClient } from '@/lib/supabase'
import { useUser } from './useUser'

export function useRealtimeProjects(organizationId: string) {
  const [projects, setProjects] = useState([])
  const { user } = useUser()
  const supabase = createClient()

  useEffect(() => {
    if (!user || !organizationId) return

    // Initial fetch
    const fetchProjects = async () => {
      const { data } = await supabase
        .from('projects')
        .select('*')
        .eq('organization_id', organizationId)
        .order('created_at', { ascending: false })
      
      if (data) setProjects(data)
    }

    fetchProjects()

    // Real-time subscription with RLS automatically applied
    const subscription = supabase
      .channel(`projects:org:${organizationId}`)
      .on(
        'postgres_changes',
        {
          event: '*',
          schema: 'public',
          table: 'projects',
          filter: `organization_id=eq.${organizationId}`
        },
        (payload) => {
          console.log('Project change:', payload)
          
          if (payload.eventType === 'INSERT') {
            setProjects(prev => [payload.new, ...prev])
          } else if (payload.eventType === 'UPDATE') {
            setProjects(prev => 
              prev.map(p => p.id === payload.new.id ? payload.new : p)
            )
          } else if (payload.eventType === 'DELETE') {
            setProjects(prev => prev.filter(p => p.id !== payload.old.id))
          }
        }
      )
      .subscribe()

    return () => {
      subscription.unsubscribe()
    }
  }, [user, organizationId, supabase])

  return { projects }
}
```