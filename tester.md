### Detailed Setup for Full-Stack Application with **React**, **Next.js**, **TypeScript**, **GraphQL**, **PostgreSQL**, and **AWS**

---

## **Frontend Setup**

### **1. Initialize Next.js with TypeScript**
```bash
npx create-next-app@latest my-app --typescript
cd my-app
pnpm install
```

### **2. Install Required Dependencies**
```bash
pnpm add @apollo/client graphql
pnpm add @graphql-codegen/cli -D
```

- **`@apollo/client`**: Apollo Client for interacting with GraphQL APIs.
- **`graphql`**: Required dependency for handling GraphQL queries.
- **`@graphql-codegen/cli`**: Optional but helpful for generating TypeScript types from your GraphQL schema.

### **3. Configure Apollo Client**

Create `lib/apolloClient.ts`:
```typescript
import { ApolloClient, InMemoryCache } from '@apollo/client';

const client = new ApolloClient({
  uri: process.env.NEXT_PUBLIC_GRAPHQL_API_URL, // Your GraphQL API endpoint
  cache: new InMemoryCache(),
});

export default client;
```

### **4. Set Up Apollo Provider**

Wrap the application with `ApolloProvider` to enable GraphQL access in components. Modify `pages/_app.tsx`:
```typescript
import { ApolloProvider } from '@apollo/client';
import client from '../lib/apolloClient';
import '../styles/globals.css';

function MyApp({ Component, pageProps }: any) {
  return (
    <ApolloProvider client={client}>
      <Component {...pageProps} />
    </ApolloProvider>
  );
}

export default MyApp;
```

### **5. Create Pages and Components**

Next.js uses a file-based routing system:
- **`pages/`** folder determines the routes.
- Add GraphQL queries to fetch data for pages.

Example: Fetch users from GraphQL API.

#### Create `pages/users.tsx`
```typescript
import { gql, useQuery } from '@apollo/client';

const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
      email
    }
  }
`;

const UsersPage = () => {
  const { data, loading, error } = useQuery(GET_USERS);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <h1>Users</h1>
      <ul>
        {data.users.map((user: any) => (
          <li key={user.id}>
            {user.name} - {user.email}
          </li>
        ))}
      </ul>
    </div>
  );
};

export default UsersPage;
```

### **6. Generate TypeScript Types for GraphQL**
Use `@graphql-codegen` to generate types for queries automatically:
```bash
npx graphql-codegen init
```

Update `codegen.yml`:
```yaml
schema: "http://localhost:4000/graphql" # Replace with your GraphQL API URL
documents: "./src/**/*.tsx"
generates:
  ./src/generated/graphql.tsx:
    plugins:
      - "typescript"
      - "typescript-operations"
      - "typescript-react-apollo"
```

Run:
```bash
pnpm graphql-codegen
```

---

## **Backend Setup**

### **1. Initialize a Node.js Project**

```bash
mkdir backend
cd backend
pnpm init -y
```

### **2. Install Backend Dependencies**
```bash
pnpm add express apollo-server-express graphql type-graphql reflect-metadata typeorm pg class-validator
pnpm add typescript ts-node-dev -D
```

- **`type-graphql`**: For building GraphQL schemas with TypeScript.
- **`typeorm`**: For database ORM.
- **`pg`**: PostgreSQL driver.

---

### **3. Configure TypeORM**
Create a `ormconfig.ts` file in the root directory:
```typescript
export default {
  type: 'postgres',
  host: process.env.DB_HOST,
  port: Number(process.env.DB_PORT),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  synchronize: true, // Auto-sync schema (turn off in production)
  logging: true,
  entities: ['src/entity/**/*.ts'],
};
```

### **4. Create a User Entity**
Create `src/entity/User.ts`:
```typescript
import { Entity, PrimaryGeneratedColumn, Column, BaseEntity } from 'typeorm';

@Entity()
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;
}
```

### **5. Create GraphQL Resolver**
Create `src/resolvers/UserResolver.ts`:
```typescript
import { Resolver, Query, Mutation, Arg } from 'type-graphql';
import { User } from '../entity/User';

@Resolver()
export class UserResolver {
  @Query(() => [User])
  async users() {
    return User.find();
  }

  @Mutation(() => User)
  async createUser(
    @Arg('name') name: string,
    @Arg('email') email: string
  ) {
    const user = User.create({ name, email });
    await user.save();
    return user;
  }
}
```

### **6. Set Up Apollo Server**
Create `src/index.ts`:
```typescript
import 'reflect-metadata';
import { ApolloServer } from 'apollo-server-express';
import Express from 'express';
import { buildSchema } from 'type-graphql';
import { createConnection } from 'typeorm';
import { UserResolver } from './resolvers/UserResolver';

(async () => {
  const app = Express();
  await createConnection();

  const schema = await buildSchema({
    resolvers: [UserResolver],
  });

  const server = new ApolloServer({ schema });
  server.applyMiddleware({ app });

  app.listen(4000, () => {
    console.log('Server started on http://localhost:4000/graphql');
  });
})();
```

Run the server:
```bash
pnpm ts-node-dev src/index.ts
```

---

## **Deploying with AWS**

### **1. PostgreSQL with AWS RDS**
1. Navigate to the **RDS Console**.
2. Create a PostgreSQL instance.
3. Note the connection details:
   - Hostname
   - Port
   - Username
   - Password
4. Update `ormconfig.ts` with these details.

### **2. Deploy Backend**
- Use **Elastic Beanstalk** or **AWS ECS** to host the Node.js backend.
- Configure environment variables for database credentials.

### **3. Deploy Frontend**
- Use **AWS Amplify** for hosting the Next.js app.
- Connect your Git repository to Amplify.
- Configure environment variables like `NEXT_PUBLIC_GRAPHQL_API_URL`.

### **4. Secure Connections**
- Use **AWS Certificate Manager** to enable HTTPS.
- Set up a load balancer in **AWS Elastic Beanstalk** for SSL termination.

---

## **Next Steps**

1. **Authentication**: Use **AWS Cognito** or **JWT** for user authentication.
2. **File Uploads**: Use **AWS S3** for storing files.
3. **Monitoring**: Use **AWS CloudWatch** for monitoring logs and metrics.
4. **Scaling**: Configure **Auto Scaling** for both frontend and backend using AWS services.

Would you like to dive deeper into any specific step?
