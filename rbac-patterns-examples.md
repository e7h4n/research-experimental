# RBAC Patterns and Best Practices 2025

## Role-Based Access Control (RBAC) Overview

Role-Based Access Control is a method of restricting network access based on the roles of individual users within an organization. In 2025, RBAC has evolved to support complex multi-tenant SaaS applications with fine-grained permissions and dynamic role assignments.

## RBAC Design Patterns

### 1. Hierarchical RBAC
```typescript
// Hierarchical role system with inheritance
export interface Role {
  id: string
  name: string
  description: string
  level: number // Higher number = more permissions
  permissions: Permission[]
  inheritsFrom?: string[] // Parent roles
}

export interface Permission {
  id: string
  resource: string // e.g., 'users', 'projects', 'billing'
  action: string   // e.g., 'read', 'write', 'delete', 'admin'
  conditions?: PermissionCondition[]
}

export interface PermissionCondition {
  field: string
  operator: 'eq' | 'neq' | 'in' | 'not_in' | 'gt' | 'lt'
  value: any
}

// Example role hierarchy
const ROLES: Role[] = [
  {
    id: 'super_admin',
    name: 'Super Administrator',
    description: 'Full system access',
    level: 100,
    permissions: [
      { id: 'all', resource: '*', action: '*' }
    ]
  },
  {
    id: 'org_admin',
    name: 'Organization Administrator',
    description: 'Full organization management',
    level: 80,
    permissions: [
      { id: 'org_read', resource: 'organization', action: 'read' },
      { id: 'org_write', resource: 'organization', action: 'write' },
      { id: 'members_admin', resource: 'members', action: '*' },
      { id: 'projects_admin', resource: 'projects', action: '*' },
      { id: 'billing_admin', resource: 'billing', action: '*' }
    ]
  },
  {
    id: 'manager',
    name: 'Manager',
    description: 'Team and project management',
    level: 60,
    inheritsFrom: ['member'],
    permissions: [
      { id: 'projects_write', resource: 'projects', action: 'write' },
      { id: 'members_read', resource: 'members', action: 'read' },
      { id: 'members_invite', resource: 'members', action: 'invite' }
    ]
  },
  {
    id: 'member',
    name: 'Member',
    description: 'Basic access',
    level: 20,
    permissions: [
      { id: 'projects_read', resource: 'projects', action: 'read' },
      { id: 'profile_write', resource: 'profile', action: 'write' }
    ]
  }
]
```

### 2. Attribute-Based Access Control (ABAC)
```typescript
// Enhanced RBAC with attributes and context
export interface AccessContext {
  user: {
    id: string
    roles: string[]
    department?: string
    location?: string
    attributes: Record<string, any>
  }
  resource: {
    type: string
    id: string
    owner?: string
    organization?: string
    attributes: Record<string, any>
  }
  environment: {
    time: Date
    ipAddress?: string
    userAgent?: string
    mfaVerified: boolean
  }
  action: string
}

export class ABACEngine {
  private policies: AccessPolicy[] = []

  addPolicy(policy: AccessPolicy) {
    this.policies.push(policy)
  }

  async evaluateAccess(context: AccessContext): Promise<AccessDecision> {
    const applicablePolicies = this.policies.filter(policy => 
      this.matchesContext(policy.conditions, context)
    )

    // Default deny
    if (applicablePolicies.length === 0) {
      return { allowed: false, reason: 'No applicable policies' }
    }

    // Check all policies (can be AND or OR logic)
    for (const policy of applicablePolicies) {
      const result = await this.evaluatePolicy(policy, context)
      if (policy.effect === 'deny' && result) {
        return { allowed: false, reason: `Denied by policy: ${policy.name}` }
      }
      if (policy.effect === 'allow' && result) {
        return { allowed: true, reason: `Allowed by policy: ${policy.name}` }
      }
    }

    return { allowed: false, reason: 'No policies granted access' }
  }

  private matchesContext(conditions: PolicyCondition[], context: AccessContext): boolean {
    return conditions.every(condition => this.evaluateCondition(condition, context))
  }

  private evaluateCondition(condition: PolicyCondition, context: AccessContext): boolean {
    const contextValue = this.getContextValue(condition.attribute, context)
    
    switch (condition.operator) {
      case 'equals':
        return contextValue === condition.value
      case 'in':
        return Array.isArray(condition.value) && condition.value.includes(contextValue)
      case 'contains':
        return Array.isArray(contextValue) && contextValue.includes(condition.value)
      case 'time_between':
        const now = context.environment.time
        return now >= new Date(condition.value.start) && now <= new Date(condition.value.end)
      default:
        return false
    }
  }

  private getContextValue(attribute: string, context: AccessContext): any {
    const parts = attribute.split('.')
    let value: any = context

    for (const part of parts) {
      if (value && typeof value === 'object') {
        value = value[part]
      } else {
        return undefined
      }
    }

    return value
  }
}

// Example ABAC policies
const timeBasedPolicy: AccessPolicy = {
  name: 'Business Hours Only',
  effect: 'deny',
  conditions: [
    {
      attribute: 'environment.time',
      operator: 'time_between',
      value: { start: '18:00', end: '08:00' }
    },
    {
      attribute: 'resource.type',
      operator: 'equals',
      value: 'sensitive_data'
    }
  ]
}

const departmentPolicy: AccessPolicy = {
  name: 'HR Access to Employee Records',
  effect: 'allow',
  conditions: [
    {
      attribute: 'user.department',
      operator: 'equals',
      value: 'hr'
    },
    {
      attribute: 'resource.type',
      operator: 'equals',
      value: 'employee_record'
    }
  ]
}
```

### 3. Multi-Tenant RBAC
```typescript
// Multi-tenant role system
export interface TenantRole {
  userId: string
  tenantId: string
  roleId: string
  grantedAt: Date
  grantedBy: string
  expiresAt?: Date
  conditions?: RoleCondition[]
}

export interface RoleCondition {
  type: 'ip_restriction' | 'time_restriction' | 'mfa_required'
  value: any
}

export class MultiTenantRBAC {
  private tenantRoles: Map<string, TenantRole[]> = new Map()

  async assignRole(assignment: {
    userId: string
    tenantId: string
    roleId: string
    grantedBy: string
    expiresAt?: Date
    conditions?: RoleCondition[]
  }): Promise<void> {
    const tenantKey = assignment.tenantId
    const roles = this.tenantRoles.get(tenantKey) || []
    
    // Remove existing role assignment for this user
    const filteredRoles = roles.filter(r => 
      !(r.userId === assignment.userId && r.roleId === assignment.roleId)
    )

    // Add new assignment
    filteredRoles.push({
      ...assignment,
      grantedAt: new Date()
    })

    this.tenantRoles.set(tenantKey, filteredRoles)
  }

  async getUserRoles(userId: string, tenantId: string): Promise<Role[]> {
    const tenantRoles = this.tenantRoles.get(tenantId) || []
    const userRoles = tenantRoles.filter(tr => tr.userId === userId)

    // Check expiration and conditions
    const validRoles = userRoles.filter(tr => {
      if (tr.expiresAt && tr.expiresAt < new Date()) {
        return false // Expired
      }

      // Check conditions
      if (tr.conditions) {
        // Implement condition checking logic
        return this.checkConditions(tr.conditions)
      }

      return true
    })

    // Fetch role details
    return this.getRoleDetails(validRoles.map(r => r.roleId))
  }

  async hasPermission(
    userId: string, 
    tenantId: string, 
    resource: string, 
    action: string
  ): Promise<boolean> {
    const userRoles = await this.getUserRoles(userId, tenantId)
    
    for (const role of userRoles) {
      for (const permission of role.permissions) {
        if (this.matchesPermission(permission, resource, action)) {
          return true
        }
      }
    }

    return false
  }

  private matchesPermission(permission: Permission, resource: string, action: string): boolean {
    // Wildcard matching
    if (permission.resource === '*' || permission.action === '*') {
      return true
    }

    // Exact matching
    if (permission.resource === resource && permission.action === action) {
      return true
    }

    // Pattern matching
    return this.matchesPattern(permission.resource, resource) &&
           this.matchesPattern(permission.action, action)
  }

  private matchesPattern(pattern: string, value: string): boolean {
    // Simple glob pattern matching
    if (pattern.includes('*')) {
      const regex = new RegExp(pattern.replace(/\*/g, '.*'))
      return regex.test(value)
    }
    return pattern === value
  }

  private checkConditions(conditions: RoleCondition[]): boolean {
    // Implement condition checking logic
    return conditions.every(condition => {
      switch (condition.type) {
        case 'mfa_required':
          // Check if MFA is verified
          return true // Placeholder
        case 'ip_restriction':
          // Check IP address
          return true // Placeholder
        case 'time_restriction':
          // Check time constraints
          return true // Placeholder
        default:
          return true
      }
    })
  }
}
```

## RBAC Implementation Strategies

### 1. Database-First RBAC (Supabase Style)
```sql
-- Complete RBAC schema for PostgreSQL
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Core tables
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  settings JSONB DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE roles (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  description TEXT,
  level INTEGER NOT NULL DEFAULT 0,
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  permissions JSONB NOT NULL DEFAULT '[]',
  is_system_role BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(name, organization_id)
);

CREATE TABLE user_roles (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  role_id UUID REFERENCES roles(id) ON DELETE CASCADE,
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  granted_by UUID REFERENCES auth.users(id),
  granted_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  expires_at TIMESTAMP WITH TIME ZONE,
  conditions JSONB DEFAULT '{}',
  UNIQUE(user_id, role_id, organization_id)
);

CREATE TABLE resources (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  type TEXT NOT NULL,
  name TEXT NOT NULL,
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  owner_id UUID REFERENCES auth.users(id),
  attributes JSONB DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Insert default system roles
INSERT INTO roles (name, description, level, permissions, is_system_role) VALUES
('super_admin', 'Super Administrator', 100, '["*:*"]', TRUE),
('admin', 'Administrator', 80, '["org:*", "users:*", "roles:*", "resources:*"]', TRUE),
('manager', 'Manager', 60, '["resources:read", "resources:write", "users:read"]', TRUE),
('member', 'Member', 20, '["resources:read"]', TRUE);

-- RLS Policies
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE roles ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_roles ENABLE ROW LEVEL SECURITY;
ALTER TABLE resources ENABLE ROW LEVEL SECURITY;

-- Organization access
CREATE POLICY "Users can view organizations they belong to" ON organizations
  FOR SELECT USING (
    id IN (
      SELECT organization_id FROM user_roles 
      WHERE user_id = auth.uid()
    )
  );

-- Resource access with RBAC
CREATE POLICY "Users can access resources based on roles" ON resources
  FOR SELECT USING (
    organization_id IN (
      SELECT ur.organization_id FROM user_roles ur
      JOIN roles r ON ur.role_id = r.id
      WHERE ur.user_id = auth.uid()
      AND (
        r.permissions @> '["resources:read"]' OR
        r.permissions @> '["*:*"]'
      )
    ) OR
    owner_id = auth.uid()
  );

-- Helper functions
CREATE OR REPLACE FUNCTION has_permission(
  user_id_param UUID,
  org_id_param UUID,
  permission_param TEXT
)
RETURNS BOOLEAN AS $$
DECLARE
  has_perm BOOLEAN := FALSE;
BEGIN
  SELECT EXISTS(
    SELECT 1 FROM user_roles ur
    JOIN roles r ON ur.role_id = r.id
    WHERE ur.user_id = user_id_param
    AND ur.organization_id = org_id_param
    AND (ur.expires_at IS NULL OR ur.expires_at > NOW())
    AND (
      r.permissions @> jsonb_build_array(permission_param) OR
      r.permissions @> '["*:*"]'
    )
  ) INTO has_perm;
  
  RETURN has_perm;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE OR REPLACE FUNCTION get_user_permissions(
  user_id_param UUID,
  org_id_param UUID
)
RETURNS JSONB AS $$
DECLARE
  permissions JSONB := '[]';
BEGIN
  SELECT jsonb_agg(DISTINCT perm)
  FROM (
    SELECT jsonb_array_elements_text(r.permissions) as perm
    FROM user_roles ur
    JOIN roles r ON ur.role_id = r.id
    WHERE ur.user_id = user_id_param
    AND ur.organization_id = org_id_param
    AND (ur.expires_at IS NULL OR ur.expires_at > NOW())
  ) perms
  INTO permissions;
  
  RETURN COALESCE(permissions, '[]');
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### 2. Application-Level RBAC (Express.js/Node.js)
```typescript
// RBAC Middleware for Express.js
import { Request, Response, NextFunction } from 'express'
import jwt from 'jsonwebtoken'

interface AuthenticatedRequest extends Request {
  user?: {
    id: string
    organizationId: string
    roles: string[]
    permissions: string[]
  }
}

export class RBACMiddleware {
  private rolePermissions: Map<string, Set<string>> = new Map()

  constructor() {
    // Initialize role permissions
    this.rolePermissions.set('admin', new Set(['*:*']))
    this.rolePermissions.set('manager', new Set([
      'users:read', 'users:write', 'projects:read', 'projects:write'
    ]))
    this.rolePermissions.set('member', new Set(['projects:read']))
  }

  // Authentication middleware
  authenticate = async (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    try {
      const token = req.headers.authorization?.split(' ')[1]
      if (!token) {
        return res.status(401).json({ error: 'No token provided' })
      }

      const decoded = jwt.verify(token, process.env.JWT_SECRET!) as any
      
      // Fetch user roles and permissions from database
      const userRoles = await this.getUserRoles(decoded.userId, decoded.organizationId)
      const permissions = this.computePermissions(userRoles)

      req.user = {
        id: decoded.userId,
        organizationId: decoded.organizationId,
        roles: userRoles,
        permissions: Array.from(permissions)
      }

      next()
    } catch (error) {
      return res.status(401).json({ error: 'Invalid token' })
    }
  }

  // Authorization middleware
  requirePermission = (requiredPermission: string) => {
    return (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
      if (!req.user) {
        return res.status(401).json({ error: 'Not authenticated' })
      }

      const hasPermission = this.checkPermission(req.user.permissions, requiredPermission)
      
      if (!hasPermission) {
        return res.status(403).json({ 
          error: 'Insufficient permissions',
          required: requiredPermission,
          available: req.user.permissions
        })
      }

      next()
    }
  }

  // Role-based middleware
  requireRole = (requiredRoles: string | string[]) => {
    const roles = Array.isArray(requiredRoles) ? requiredRoles : [requiredRoles]
    
    return (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
      if (!req.user) {
        return res.status(401).json({ error: 'Not authenticated' })
      }

      const hasRole = roles.some(role => req.user!.roles.includes(role))
      
      if (!hasRole) {
        return res.status(403).json({ 
          error: 'Insufficient role',
          required: roles,
          available: req.user.roles
        })
      }

      next()
    }
  }

  // Resource ownership middleware
  requireOwnership = (resourceParam: string = 'id') => {
    return async (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
      if (!req.user) {
        return res.status(401).json({ error: 'Not authenticated' })
      }

      const resourceId = req.params[resourceParam]
      const resource = await this.getResource(resourceId)

      if (!resource) {
        return res.status(404).json({ error: 'Resource not found' })
      }

      // Check if user owns the resource or has admin permissions
      const isOwner = resource.ownerId === req.user.id
      const isAdmin = this.checkPermission(req.user.permissions, '*:*')

      if (!isOwner && !isAdmin) {
        return res.status(403).json({ error: 'Not authorized to access this resource' })
      }

      next()
    }
  }

  private checkPermission(userPermissions: string[], requiredPermission: string): boolean {
    // Check for wildcard permissions
    if (userPermissions.includes('*:*')) {
      return true
    }

    // Check for exact match
    if (userPermissions.includes(requiredPermission)) {
      return true
    }

    // Check for resource wildcard (e.g., users:* matches users:read)
    const [resource, action] = requiredPermission.split(':')
    if (userPermissions.includes(`${resource}:*`)) {
      return true
    }

    return false
  }

  private computePermissions(roles: string[]): Set<string> {
    const permissions = new Set<string>()
    
    for (const role of roles) {
      const rolePermissions = this.rolePermissions.get(role)
      if (rolePermissions) {
        rolePermissions.forEach(perm => permissions.add(perm))
      }
    }

    return permissions
  }

  private async getUserRoles(userId: string, organizationId: string): Promise<string[]> {
    // Database query to get user roles
    // This would connect to your database
    return ['member'] // Placeholder
  }

  private async getResource(resourceId: string) {
    // Database query to get resource details
    return null // Placeholder
  }
}

// Usage in Express routes
const rbac = new RBACMiddleware()

app.use('/api', rbac.authenticate)

// Require specific permission
app.get('/api/users', 
  rbac.requirePermission('users:read'),
  (req, res) => {
    res.json({ users: [] })
  }
)

// Require specific role
app.post('/api/users',
  rbac.requireRole(['admin', 'manager']),
  (req, res) => {
    res.json({ message: 'User created' })
  }
)

// Require resource ownership
app.put('/api/projects/:id',
  rbac.requireOwnership('id'),
  (req, res) => {
    res.json({ message: 'Project updated' })
  }
)
```

### 3. React Components with RBAC
```typescript
// React RBAC components and hooks
import { createContext, useContext, ReactNode } from 'react'

interface RBACContextValue {
  user: {
    id: string
    roles: string[]
    permissions: string[]
  } | null
  hasPermission: (permission: string) => boolean
  hasRole: (role: string | string[]) => boolean
  hasAnyRole: (roles: string[]) => boolean
  isOwner: (resourceOwnerId: string) => boolean
}

const RBACContext = createContext<RBACContextValue | undefined>(undefined)

export function RBACProvider({ children, user }: { 
  children: ReactNode
  user: RBACContextValue['user']
}) {
  const hasPermission = (permission: string): boolean => {
    if (!user?.permissions) return false
    
    // Check wildcard
    if (user.permissions.includes('*:*')) return true
    
    // Check exact match
    if (user.permissions.includes(permission)) return true
    
    // Check resource wildcard
    const [resource] = permission.split(':')
    return user.permissions.includes(`${resource}:*`)
  }

  const hasRole = (role: string | string[]): boolean => {
    if (!user?.roles) return false
    const roles = Array.isArray(role) ? role : [role]
    return roles.some(r => user.roles.includes(r))
  }

  const hasAnyRole = (roles: string[]): boolean => {
    if (!user?.roles) return false
    return roles.some(role => user.roles.includes(role))
  }

  const isOwner = (resourceOwnerId: string): boolean => {
    return user?.id === resourceOwnerId
  }

  return (
    <RBACContext.Provider value={{ 
      user, 
      hasPermission, 
      hasRole, 
      hasAnyRole, 
      isOwner 
    }}>
      {children}
    </RBACContext.Provider>
  )
}

export function useRBAC() {
  const context = useContext(RBACContext)
  if (context === undefined) {
    throw new Error('useRBAC must be used within an RBACProvider')
  }
  return context
}

// Permission gate component
interface PermissionGateProps {
  permission?: string
  role?: string | string[]
  ownerId?: string
  fallback?: ReactNode
  children: ReactNode
}

export function PermissionGate({ 
  permission, 
  role, 
  ownerId, 
  fallback = null, 
  children 
}: PermissionGateProps) {
  const { hasPermission, hasRole, isOwner } = useRBAC()

  let hasAccess = true

  // Check permission
  if (permission && !hasPermission(permission)) {
    hasAccess = false
  }

  // Check role
  if (role && !hasRole(role)) {
    hasAccess = false
  }

  // Check ownership
  if (ownerId && !isOwner(ownerId)) {
    hasAccess = false
  }

  return hasAccess ? <>{children}</> : <>{fallback}</>
}

// Hook for conditional rendering
export function usePermission(permission: string) {
  const { hasPermission } = useRBAC()
  return hasPermission(permission)
}

export function useRole(role: string | string[]) {
  const { hasRole } = useRBAC()
  return hasRole(role)
}

// Usage examples
function UserManagement() {
  const canManageUsers = usePermission('users:write')
  
  return (
    <div>
      <h1>Users</h1>
      
      <PermissionGate permission="users:read">
        <UserList />
      </PermissionGate>
      
      <PermissionGate 
        role={['admin', 'manager']}
        fallback={<p>You don't have permission to manage users</p>}
      >
        <button>Add User</button>
        <button>Edit User</button>
      </PermissionGate>

      {canManageUsers && (
        <button>Bulk Actions</button>
      )}
    </div>
  )
}

function ProjectCard({ project }: { project: { id: string, ownerId: string, name: string } }) {
  return (
    <div className="border rounded p-4">
      <h3>{project.name}</h3>
      
      <PermissionGate ownerId={project.ownerId}>
        <button>Edit</button>
      </PermissionGate>
      
      <PermissionGate 
        permission="projects:delete"
        ownerId={project.ownerId}
      >
        <button>Delete</button>
      </PermissionGate>
    </div>
  )
}
```

## RBAC Best Practices 2025

### 1. Principle of Least Privilege
- **Default Deny**: Start with no permissions and explicitly grant access
- **Time-Limited Roles**: Use expiration dates for temporary access
- **Just-in-Time Access**: Grant elevated permissions only when needed

### 2. Separation of Concerns
- **Role Definition**: Separate role creation from permission assignment
- **Audit Trail**: Log all role changes and permission grants
- **Regular Reviews**: Periodic access reviews and cleanup

### 3. Scalable Architecture
- **Caching Strategy**: Cache permission checks for performance
- **Event-Driven Updates**: Real-time permission updates across services
- **Microservice Integration**: Distributed RBAC across service boundaries

### 4. Security Considerations
- **Token-Based**: Use JWTs with embedded permissions for stateless auth
- **Rate Limiting**: Prevent permission enumeration attacks
- **Secure Defaults**: Safe configuration out of the box

This comprehensive RBAC system provides the foundation for secure, scalable access control in modern SaaS applications.