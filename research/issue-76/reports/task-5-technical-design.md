# Technical Design Document: AI Video Creation Agent

## 1. System Overview

### Project Description
An AI-powered web application that enables users to create videos through conversational interaction, leveraging Google's AI ecosystem (Gemini, Imagen 3, and Veo 3) to generate images and videos from text descriptions.

### Key Features
- Google OAuth authentication
- Conversational UI for video requirements
- Multi-stage content generation (script → images → video)
- Real-time progress tracking
- Video preview and download
- Cloud-based processing

## 2. Technical Architecture

### High-Level Architecture

```mermaid
graph TB
    subgraph "Frontend - Next.js 14"
        A[Landing Page]
        B[Dashboard]
        C[Video Creator Wizard]
        D[Gallery]
    end
    
    subgraph "Authentication Layer"
        E[NextAuth.js]
        F[Google OAuth Provider]
    end
    
    subgraph "API Layer - Next.js API Routes"
        G[/api/auth/*]
        H[/api/generate/script]
        I[/api/generate/images]
        J[/api/generate/video]
        K[/api/projects/*]
    end
    
    subgraph "AI Services"
        L[Gemini 1.5 Flash]
        M[Imagen 3]
        N[Veo 3]
    end
    
    subgraph "Storage & Database"
        O[Cloud Storage]
        P[PostgreSQL/Supabase]
        Q[Redis Cache]
    end
    
    subgraph "Infrastructure"
        R[Google Cloud Run]
        S[Cloud CDN]
    end
    
    A --> E
    B --> G
    C --> H
    C --> I
    C --> J
    H --> L
    I --> M
    J --> N
    K --> P
    J --> O
    O --> S
    R --> A
```

### Data Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant Gemini
    participant Imagen
    participant Veo
    participant Storage
    
    User->>Frontend: Describe video concept
    Frontend->>API: POST /api/generate/script
    API->>Gemini: Generate script & scenes
    Gemini-->>API: Return structured content
    API-->>Frontend: Display script preview
    
    Frontend->>API: POST /api/generate/images
    API->>Imagen: Generate key frames
    Imagen-->>API: Return image URLs
    API->>Storage: Save images
    API-->>Frontend: Display images
    
    Frontend->>API: POST /api/generate/video
    API->>Veo: Create video from images
    Veo-->>API: Return video URL
    API->>Storage: Save video
    API-->>Frontend: Display video player
```

## 3. Technology Stack

### Frontend
- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **UI Components**: shadcn/ui
- **State Management**: Zustand
- **Forms**: React Hook Form + Zod
- **Animations**: Framer Motion

### Backend
- **Runtime**: Node.js 20
- **Framework**: Next.js API Routes
- **Authentication**: NextAuth.js v5
- **Database ORM**: Prisma
- **Queue Management**: BullMQ (Redis)
- **File Upload**: Multer/FormData

### AI Services
- **LLM**: Gemini 1.5 Flash (via Gemini API)
- **Image Generation**: Imagen 3 (imagen-3.0-generate-002)
- **Video Generation**: Veo 3 (veo-3.0-generate-preview)

### Infrastructure
- **Deployment**: Google Cloud Run
- **Database**: PostgreSQL (Cloud SQL) or Supabase
- **Cache**: Redis (Memory Store)
- **Storage**: Google Cloud Storage
- **CDN**: Cloud CDN
- **Monitoring**: Cloud Operations Suite

## 4. Database Schema

```sql
-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    google_id VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255),
    avatar_url TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Projects table
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(50) DEFAULT 'draft',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Scripts table
CREATE TABLE scripts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id),
    content JSONB NOT NULL,
    prompt TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Images table
CREATE TABLE images (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id),
    url TEXT NOT NULL,
    prompt TEXT,
    type VARCHAR(50), -- 'character', 'scene', 'background'
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Videos table
CREATE TABLE videos (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id),
    url TEXT NOT NULL,
    duration INTEGER,
    resolution VARCHAR(20),
    status VARCHAR(50),
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);
```

## 5. API Specification

### Authentication Endpoints

```typescript
// Google OAuth callback
GET /api/auth/callback/google

// Session management
GET /api/auth/session
POST /api/auth/signout
```

### Generation Endpoints

```typescript
// Script generation
POST /api/generate/script
Request: {
  prompt: string;
  style?: string;
  duration?: number;
}
Response: {
  script: {
    scenes: Array<{
      id: string;
      description: string;
      dialogue?: string;
      duration: number;
    }>;
    characters: Array<{
      name: string;
      description: string;
    }>;
  }
}

// Image generation
POST /api/generate/images
Request: {
  projectId: string;
  scenes: Array<{
    id: string;
    prompt: string;
  }>;
}
Response: {
  images: Array<{
    sceneId: string;
    url: string;
  }>;
}

// Video generation
POST /api/generate/video
Request: {
  projectId: string;
  images: string[];
  script: object;
  options: {
    resolution: '720p' | '1080p';
    aspectRatio: '16:9' | '9:16';
  }
}
Response: {
  videoId: string;
  status: 'processing' | 'completed';
  url?: string;
}
```

## 6. Implementation Details

### Phase 1: Authentication & Setup (Hours 0-2)

```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import GoogleProvider from 'next-auth/providers/google';

export const authOptions = {
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
  callbacks: {
    async session({ session, token }) {
      return session;
    },
  },
};

const handler = NextAuth(authOptions);
export { handler as GET, handler as POST };
```

### Phase 2: Script Generation (Hours 2-6)

```typescript
// app/api/generate/script/route.ts
import { GoogleGenerativeAI } from '@google/generative-ai';

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY!);

export async function POST(request: Request) {
  const { prompt } = await request.json();
  
  const model = genAI.getGenerativeModel({ 
    model: "gemini-1.5-flash" 
  });
  
  const systemPrompt = `
    Generate a video script with the following structure:
    1. Scene descriptions (visual details)
    2. Character descriptions
    3. Dialogue (if any)
    4. Camera movements
    Return as structured JSON.
  `;
  
  const result = await model.generateContent(
    systemPrompt + "\n\n" + prompt
  );
  
  return Response.json({
    script: JSON.parse(result.response.text())
  });
}
```

### Phase 3: Image Generation (Hours 6-10)

```typescript
// app/api/generate/images/route.ts
import { GoogleGenerativeAI } from '@google/generative-ai';

export async function POST(request: Request) {
  const { scenes } = await request.json();
  const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY!);
  
  const imagePromises = scenes.map(async (scene: any) => {
    const response = await genAI.models.generateImages({
      model: 'imagen-3.0-generate-002',
      prompt: scene.prompt,
      config: {
        numberOfImages: 1,
        aspectRatio: '16:9'
      }
    });
    
    return {
      sceneId: scene.id,
      url: response.images[0].url
    };
  });
  
  const images = await Promise.all(imagePromises);
  return Response.json({ images });
}
```

### Phase 4: Video Generation (Hours 10-14)

```typescript
// app/api/generate/video/route.ts
import { VertexAI } from '@google-cloud/vertexai';

const vertex = new VertexAI({
  project: process.env.GCP_PROJECT_ID!,
  location: 'us-central1'
});

export async function POST(request: Request) {
  const { images, script } = await request.json();
  
  const model = vertex.preview.getGenerativeModel({
    model: 'veo-3.0-generate-preview'
  });
  
  const videoRequest = {
    prompt: script.summary,
    images: images,
    config: {
      duration: 10, // seconds
      resolution: '1080p',
      aspectRatio: '16:9'
    }
  };
  
  const response = await model.generateVideo(videoRequest);
  
  return Response.json({
    videoId: response.name,
    status: 'processing'
  });
}
```

### Phase 5: Frontend Implementation (Hours 14-18)

```typescript
// app/create/page.tsx
'use client';

import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { Progress } from '@/components/ui/progress';

export default function CreatePage() {
  const [step, setStep] = useState(1);
  const [progress, setProgress] = useState(0);
  const { register, handleSubmit } = useForm();
  
  const onSubmit = async (data: any) => {
    // Step 1: Generate script
    setProgress(20);
    const scriptRes = await fetch('/api/generate/script', {
      method: 'POST',
      body: JSON.stringify(data)
    });
    
    // Step 2: Generate images
    setProgress(50);
    const imagesRes = await fetch('/api/generate/images', {
      method: 'POST',
      body: JSON.stringify(await scriptRes.json())
    });
    
    // Step 3: Generate video
    setProgress(80);
    const videoRes = await fetch('/api/generate/video', {
      method: 'POST',
      body: JSON.stringify(await imagesRes.json())
    });
    
    setProgress(100);
  };
  
  return (
    <div className="container mx-auto p-8">
      <Progress value={progress} />
      <form onSubmit={handleSubmit(onSubmit)}>
        {/* Form implementation */}
      </form>
    </div>
  );
}
```

## 7. Deployment Configuration

### Dockerfile

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000
ENV PORT 3000
CMD ["node", "server.js"]
```

### Cloud Run Configuration

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: video-agent
spec:
  template:
    metadata:
      annotations:
        run.googleapis.com/execution-environment: gen2
    spec:
      containerConcurrency: 100
      timeoutSeconds: 300
      containers:
      - image: gcr.io/PROJECT_ID/video-agent
        ports:
        - containerPort: 3000
        resources:
          limits:
            cpu: '2'
            memory: '2Gi'
        env:
        - name: NODE_ENV
          value: production
```

## 8. Environment Variables

```env
# Authentication
GOOGLE_CLIENT_ID=xxx
GOOGLE_CLIENT_SECRET=xxx
NEXTAUTH_URL=https://your-domain.com
NEXTAUTH_SECRET=xxx

# Google AI APIs
GEMINI_API_KEY=xxx
GCP_PROJECT_ID=xxx
VERTEX_AI_LOCATION=us-central1

# Database
DATABASE_URL=postgresql://user:pass@host:5432/db

# Storage
GCS_BUCKET=your-bucket-name

# Redis
REDIS_URL=redis://localhost:6379
```

## 9. Cost Estimation

### API Costs (per 1000 generations)

| Service | Unit Cost | Monthly Est. |
|---------|-----------|--------------|
| Gemini 1.5 Flash | $0.075/1M tokens | ~$10 |
| Imagen 3 | $0.03/image | $30 |
| Veo 3 | $0.75/second | $750 |
| Cloud Run | $0.00002/vCPU-second | ~$5 |
| Cloud Storage | $0.02/GB | ~$2 |

### Free Tier Benefits
- Cloud Run: 2M requests/month free
- Gemini API: Free tier available
- Cloud Storage: 5GB free

## 10. MVP Deliverables

### Core Features (Must Have)
- [x] Google authentication
- [x] Text input for video concept
- [x] Script generation with Gemini
- [x] Image generation with Imagen
- [x] Video creation with Veo
- [x] Download functionality

### Extended Features (If Time Permits)
- [ ] Gallery view
- [ ] Video editing capabilities
- [ ] Multiple style options
- [ ] Social sharing
- [ ] Usage analytics

## 11. Development Timeline

```mermaid
gantt
    title 24-Hour Development Schedule
    dateFormat HH:mm
    
    section Setup
    Environment Setup     :00:00, 1h
    Auth Implementation   :01:00, 2h
    
    section Backend
    API Structure        :03:00, 2h
    Gemini Integration   :05:00, 2h
    Imagen Integration   :07:00, 2h
    Veo Integration      :09:00, 3h
    
    section Frontend
    UI Components        :12:00, 3h
    Form Wizard          :15:00, 2h
    Progress Display     :17:00, 1h
    
    section Testing
    Integration Testing  :18:00, 2h
    Bug Fixes           :20:00, 2h
    
    section Deployment
    Deploy to Cloud Run  :22:00, 1h
    Final Testing        :23:00, 1h
```

## 12. Success Metrics

- User can complete full video creation flow
- Video generation success rate > 80%
- Response time < 30 seconds per stage
- Application handles concurrent users
- Clean, intuitive UI/UX