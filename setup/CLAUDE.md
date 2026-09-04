# CLAUDE.md — Conduit Codebase Guide

Branch: `rwa/design-and-implementation-v2`

## What this app is

**Conduit** — the reference implementation of the **RealWorld** spec
(github.com/gothinkster/realworld), i.e. a **Medium.com clone**: a social blogging platform.
Authenticated users publish/read/edit/delete **articles** (title, description, body, tags),
**comment** on them, **favorite** them, and **follow** authors; articles are keyed by a URL
**slug**. The frontend (`frontend/`, React SPA) calls the backend (`backend/`, NestJS API at
`/api`, Swagger at `/docs`) over HTTP; the backend persists to MySQL via MikroORM. No tests,
no Storybook, no Postman collection — drive the API via Swagger or curl.

---

## Stack and Exact Versions

**Backend** (`backend/package.json`):
- Node.js (runtime; version determined by devcontainer image)
- NestJS `^10.0.5` (`@nestjs/common`, `@nestjs/core`, `@nestjs/platform-express`)
- MikroORM `5.7.14` (`@mikro-orm/core`, `@mikro-orm/mysql`, `@mikro-orm/migrations`, `@mikro-orm/nestjs 5.2.0`, `@mikro-orm/reflection`, `@mikro-orm/seeder`)
- TypeScript `5.1.6`
- `@nestjs/swagger ^7.1.1` + `swagger-ui-express ^5.0.0`
- `class-validator ^0.14.0` + `class-transformer ^0.5.1`
- `jsonwebtoken ^9.0.1` + `passport-jwt ^4.0.1`
- `slug ^8.2.2` (for article slugs)
- `crypto-js ^4.1.1`
- Dev server via `ts-node-dev --respawn`

**Frontend** (`frontend/package.json`):
- React `^18.2.0` + ReactDOM
- TypeScript `^5.3.3`
- Vite `^5.1.4` + `@vitejs/plugin-react ^4.2.1`
- Redux Toolkit `^2.2.1` + react-redux `^9.1.0`
- axios `^1.6.7`
- `decoders ^2.0.1` (runtime response validation/decoding)
- `@hqoss/monads ^0.5.0` (Result type for error handling)
- `ramda ^0.29.1`
- `react-router-dom ^6.22.2`
- `date-fns ^3.3.1`

**Root** (`package.json`):
- Workspace orchestration via `npm-run-all 4.1.5`
- Submit script via `tsx ^4.19.3`
- MySQL `8.1` (Docker image in devcontainer)

---

## Top-Level Layout

```
/
├── backend/                  NestJS API server
│   ├── src/                  Application source
│   ├── mikro-orm.config.ts   ORM configuration (DB connection, entities, migrations)
│   ├── nestconfig.json       Entry file pointer (src/main.ts)
│   ├── nest-cli.json         NestJS CLI config
│   └── package.json
├── frontend/                 React SPA
│   ├── src/                  Application source
│   ├── vite.config.ts        Vite dev server (port 3001, /api proxy to :3000)
│   └── package.json
├── .devcontainer/            GitHub Codespaces/Docker dev environment
│   ├── devcontainer.json     VSCode extensions (Cline 3.34.0, MySQL client, ESLint, Prettier)
│   └── docker-compose.yml    App container + MySQL 8.1 container
├── .vscode/tasks.json        Auto-start tasks: install, start backend, start frontend, seed
├── submission/               Drop screenshots + patch here before submitting (gitignored)
├── submit.ts                 Submission script (creates git diff patch + zip, uploads)
├── plan.md                   Plan template (candidate fills this before coding)
├── diagram.png               AWS production architecture
└── package.json              Root workspace scripts
```

### `backend/src/` Structure

```
src/
├── main.ts                   Bootstrap: NestFactory, global prefix /api, Swagger at /docs, port 3000
├── app.module.ts             Root module: MikroORM init, MikroOrmMiddleware on all routes, auto-migrate on start
├── app.controller.ts         Health check GET /api -> "Hello World!"
├── config.ts                 JWT secret constant (SECRET = 'secret-key')
├── article/
│   ├── article.entity.ts     Article entity (id, slug, title, description, body, createdAt, updatedAt, tagList, author, comments, favoritesCount)
│   ├── comment.entity.ts     Comment entity (id, createdAt, updatedAt, body, article, author)
│   ├── article.module.ts     Module: registers Article/Comment/User entities, AuthMiddleware on protected routes
│   ├── article.service.ts    Business logic for articles and comments
│   ├── article.controller.ts REST endpoints for articles, comments, favorites
│   ├── article.interface.ts  IArticleRO, IArticlesRO, ICommentsRO
│   └── dto/
│       ├── create-article.dto.ts   CreateArticleDto (title, description, body, tagList)
│       ├── create-comment.ts       CreateCommentDto (body)
│       └── index.ts
├── user/
│   ├── user.entity.ts        User entity (id, username, email, bio, image, password[hidden], favorites, followers, followed, articles)
│   ├── user.repository.ts    UserRepository extends EntityRepository<User>
│   ├── user.module.ts        Module: registers User entity, exports UserService, AuthMiddleware on GET/PUT /user
│   ├── user.service.ts       Business logic: create, login, findById, findByEmail, update, generateJWT, findAllWithPagination
│   ├── user.controller.ts    GET /user, PUT /user, POST /users, POST /users/login, GET /users, DELETE /users/:slug
│   ├── user.decorator.ts     @User() param decorator: extracts user from req.user or decodes JWT header
│   ├── auth.middleware.ts    AuthMiddleware: validates Bearer token, populates req.user
│   ├── user.interface.ts     IUserData, IUserRO
│   └── dto/
│       ├── create-user.dto.ts   CreateUserDto (username, email, password — all @IsNotEmpty)
│       ├── login-user.dto.ts    LoginUserDto (email, password — all @IsNotEmpty)
│       ├── update-user.dto.ts   UpdateUserDto (bio, email, image, username — no validators)
│       └── index.ts
├── profile/
│   ├── profile.module.ts     Module: imports UserModule, registers AuthMiddleware on follow/unfollow
│   ├── profile.service.ts    findProfile, follow, unFollow
│   ├── profile.controller.ts GET /profiles/:username, POST/DELETE /profiles/:username/follow
│   └── profile.interface.ts  IProfileData, IProfileRO
├── tag/
│   ├── tag.entity.ts         Tag entity (id, tag)
│   ├── tag.module.ts         Module
│   ├── tag.service.ts        findAll -> { tags: string[] }
│   ├── tag.controller.ts     GET /tags
│   └── tag.interface.ts      ITagsRO
├── shared/
│   └── pipes/
│       └── validation.pipe.ts   ValidationPipe: plainToClass + class-validator, returns structured errors
├── migrations/
│   └── InitialMigration.ts   Single migration: creates user, user_to_follower, tag, article, comment, user_favorites tables
└── seeders/
    └── database.seeder.ts    DatabaseSeeder: skips if User table non-empty; creates 3 users + 4 tags + 2 articles
```

### `frontend/src/` Structure

```
src/
├── index.tsx                 React entry: renders <App />
├── index.css / conduit.css   Global styles
├── env.d.ts                  Vite env type declarations
├── config/
│   └── settings.ts           baseApiUrl: '/api/'
├── state/
│   ├── store.ts              Redux store: combines all slices
│   └── storeHooks.ts         useStore, useStoreWithInitializer hooks
├── services/
│   └── conduit.ts            All API calls (axios, base url /api/): getArticles, login, createArticle, updateArticle, etc.
├── types/
│   ├── article.ts            Article, ArticleForEditor, MultipleArticles, decoders, filter interfaces
│   ├── comment.ts            Comment, commentDecoder
│   ├── error.ts              GenericErrors = Record<string, string[]>
│   ├── genericFormField.tsx  GenericFormField type, buildGenericFormField helper
│   ├── location.ts           redirect() helper (sets location.hash)
│   ├── object.ts             objectToQueryString
│   ├── profile.ts            Profile, profileDecoder
│   ├── style.ts              classObjectToClassName
│   └── user.ts               User, PublicUser, UserSettings, userDecoder, loadUserIntoApp
└── components/
    ├── App/
    │   ├── App.tsx           Root: HashRouter, route definitions, initial auth load
    │   └── App.slice.ts      State: user, loading; actions: loadUser, logout, endLoad
    ├── Pages/
    │   ├── Home/             Home page (feed tabs, tag sidebar)
    │   ├── Login/            Login form
    │   ├── Register/         Registration form
    │   ├── Settings/         User settings form
    │   ├── ArticlePage/      Full article view (comments, favorite, follow)
    │   ├── NewArticle/       Create article page (wraps ArticleEditor)
    │   ├── EditArticle/      Edit article page (wraps ArticleEditor)
    │   └── ProfilePage/      User profile page
    ├── ArticleEditor/        Shared editor form component + slice
    ├── ArticlesViewer/       Article list with tabs + pagination + slice
    ├── ArticlePreview/       Single article card in list
    ├── ContainerPage/        Layout wrapper
    ├── Errors/               Error display component
    ├── Footer/               Site footer
    ├── FormGroup/            Form field wrapper
    ├── GenericForm/          Reusable form component
    ├── Header/               Nav header
    ├── Pagination/           Page controls
    └── UserInfo/             Author info component
```

---

## Backend Architecture

### Module Organization

Each domain (user, article, profile, tag) is a NestJS **feature module** containing:
- `*.entity.ts` — MikroORM entity class
- `*.module.ts` — registers entities via `MikroOrmModule.forFeature()`, declares controllers/providers, configures middleware
- `*.service.ts` — `@Injectable()` class with business logic, injects EntityManager and repositories
- `*.controller.ts` — `@Controller()` class with route handlers, injects service
- `dto/` — plain TS classes with class-validator decorators for incoming request bodies

**AppModule** (`src/app.module.ts`) is the root: imports `MikroOrmModule.forRoot(ormConfig)` and all feature modules. It implements `OnModuleInit` to run pending migrations at startup (`this.orm.getMigrator().up()`). It also registers `MikroOrmMiddleware` on all routes (`*`) to establish the per-request EntityManager context.

### Request Lifecycle

1. HTTP request arrives at port 3000 (`/api/...`)
2. `MikroOrmMiddleware` (registered globally by AppModule) creates a per-request `EntityManager` fork — this is the MikroORM Unit of Work context
3. `AuthMiddleware` (registered per-module on protected routes) validates the `Authorization: Token <jwt>` header, verifies the JWT, loads the user from DB, and sets `req.user`
4. NestJS routes to the controller method
5. `@User()` param decorator extracts `req.user` (if populated by middleware) or decodes the JWT directly from the header (for optional-auth routes)
6. Controller calls service method
7. Service uses injected repositories / EntityManager to query/mutate the database
8. Service calls `entity.toJSON(user)` to serialize with computed fields (favorited, following)
9. Controller returns the response object (NestJS auto-serializes to JSON)

---

## Complete End-to-End Feature: Create Article

This traces `POST /api/articles` from entity to React render.

### 1. Entity — `backend/src/article/article.entity.ts`
- `@Entity()` class `Article` with MikroORM decorators
- `tagList` is stored as `ArrayType` (serialized as comma-separated text in MySQL column `tag_list`)
- `comments` is a `Collection<Comment>` with `eager: true` and `orphanRemoval: true`
- `author` is a `@ManyToOne(() => User)` stored as `author_id` foreign key
- Constructor: `new Article(author, title, description, body)` — generates `slug` using `slug(title)` + random suffix
- `toJSON(user?)` — calls `wrap(this).toObject()`, adds computed `favorited` and replaces `author` with `author.toJSON(user)`

### 2. Migration — `backend/src/migrations/InitialMigration.ts`
- `article` table DDL: `id`, `slug`, `title`, `description`, `body` (all `varchar(255)`), `created_at`, `updated_at` (datetime), `tag_list` (text), `author_id` (FK to user), `favorites_count` (int)
- **NOTE**: `body` and `description` columns are `varchar(255)` — this is a length constraint on the existing schema.
- Migrations are listed explicitly in `mikro-orm.config.ts` under `migrationsList` (not auto-discovered from files). Adding a migration requires adding it to that array.

### 3. Repository
Article uses the generic `EntityRepository<Article>` injected via `@InjectRepository(Article)` — no custom repository class (unlike `User` which has `UserRepository`).

### 4. Service — `backend/src/article/article.service.ts` → `create(userId, dto)`
```
const user = await this.userRepository.findOne({ id: userId }, { populate: ['followers', 'favorites', 'articles'] });
const article = new Article(user!, dto.title, dto.description, dto.body);
article.tagList.push(...dto.tagList);
user?.articles.add(article);
await this.em.flush();
return { article: article.toJSON(user!) };
```
- Uses `this.em.flush()` to persist (no explicit `persist()` call — article is tracked because it's added to `user.articles` collection)

### 5. Controller — `backend/src/article/article.controller.ts` → `create`
```typescript
@Post()
async create(@User('id') userId: number, @Body('article') articleData: CreateArticleDto) {
  return this.articleService.create(userId, articleData);
}
```
- Route: `POST /api/articles`
- `@User('id')` extracts the `id` field from the decoded JWT / req.user
- `@Body('article')` unwraps the `{ article: {...} }` envelope

### 6. DTO — `backend/src/article/dto/create-article.dto.ts`
```typescript
export class CreateArticleDto {
  readonly title: string;
  readonly description: string;
  readonly body: string;
  readonly tagList: string[];
}
```
No class-validator decorators on CreateArticleDto — validation is not enforced server-side for articles (contrast with `CreateUserDto` which uses `@IsNotEmpty()`).

### 7. Route and OpenAPI
- Global prefix `api` set in `main.ts` → full path: `POST /api/articles`
- `@ApiTags('articles')`, `@ApiOperation`, `@ApiResponse` decorators on controller → visible at `http://localhost:3000/docs`
- `@ApiBearerAuth()` on controller class → Swagger shows lock icon

### 8. Auth Guard (Middleware)
- `AuthMiddleware` registered in `ArticleModule.configure()` for `POST /articles`
- Any request without a valid `Authorization: Token <jwt>` header is rejected with 401

### 9. Frontend API Call — `frontend/src/services/conduit.ts` → `createArticle`
```typescript
export async function createArticle(article: ArticleForEditor): Promise<Result<Article, GenericErrors>> {
  try {
    const { data } = await axios.post('articles', { article });
    return Ok(object({ article: articleDecoder }).verify(data).article);
  } catch (error) {
    const axiosError = error as AxiosError;
    return Err(object({ errors: genericErrorsDecoder }).verify(axiosError.response?.data).errors);
  }
}
```
- axios base URL is `/api/` (from `settings.ts`) — Vite dev proxy forwards `/api` to `http://localhost:3000/`
- Response is validated at runtime with `decoders` library (`articleDecoder`)
- Returns `Result<Article, GenericErrors>` (monad from `@hqoss/monads`)

### 10. Frontend Page — `frontend/src/components/Pages/NewArticle/NewArticle.tsx`
- Dispatches `initializeEditor()` on mount
- Renders `<ArticleEditor onSubmit={onSubmit} />`
- `onSubmit` calls `createArticle(store.getState().editor.article)`, on success navigates to `#/article/:slug`

### 11. Editor Component — `frontend/src/components/ArticleEditor/ArticleEditor.tsx`
- Reads state from `editor` slice
- Renders `<GenericForm>` with fields: title, description, body, tag (list input)
- Dispatches `updateField`, `addTag`, `removeTag` actions

### 12. Redux Slice — `frontend/src/components/ArticleEditor/ArticleEditor.slice.tsx`
- State: `{ article: ArticleForEditor, tag: string, submitting: boolean, errors: GenericErrors, loading: boolean }`
- `ArticleForEditor = { title, description, body, tagList }`

---

## Data Model

### Entities

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `User` | id, username, email (hidden), bio, image, password (hidden, SHA-256 hex) | favorites: M2M Article; followers/followed: M2M User (self-ref, pivot: user_to_follower); articles: O2M Article |
| `Article` | id, slug, title, description, body, createdAt, updatedAt, tagList (ArrayType), favoritesCount | author: M2O User; comments: O2M Comment (eager, orphanRemoval) |
| `Comment` | id, createdAt, updatedAt, body | article: M2O Article; author: M2O User |
| `Tag` | id, tag | (no relations) |

### Database Tables (from InitialMigration)
- `user` — 6 columns, no explicit unique constraints in DDL
- `user_to_follower` — pivot (follower int, following int), composite PK
- `tag` — 2 columns
- `article` — 11 columns; FK article.author_id → user.id (cascade update)
- `comment` — 6 columns; FK comment.article_id → article.id (cascade update); FK comment.author_id → user.id
- `user_favorites` — pivot (user_id, article_id), composite PK, cascade delete

### Migrations
- Location: `backend/src/migrations/`
- **Explicitly listed** in `mikro-orm.config.ts` under `migrationsList: [{ name, class }]` — NOT auto-discovered
- To add a migration: create `backend/src/migrations/YourMigration.ts` implementing `Migration`, then add it to `migrationsList` in `mikro-orm.config.ts`
- MikroORM runs `migrator.up()` automatically on every app start (in `AppModule.onModuleInit`)

### Seeds — `backend/src/seeders/database.seeder.ts`
- Entrypoint: `DatabaseSeeder` (class Seeder)
- **Guard**: checks `em.count(User) > 0` — if any users exist, seed is skipped entirely
- Creates: 3 users (john/bennie/zolly, password `password`), 4 tags (coding/javascript/angular/react), 2 articles
- Seed command: `npm run seed` (from backend dir) or `npm run seed` from root
- **Important**: seeder runs as a separate CLI command, NOT automatically on app start. The VSCode task "Run Seed" runs it after the backend is up.

---

## Auth: JWT Login End-to-End

### Backend
1. `POST /api/users/login` → `UserController.login(loginUserDto: LoginUserDto)`
2. `UserService.findOne({ email, password: sha256(password) })` — password compared as SHA-256 hex
3. If user not found → 401
4. `UserService.generateJWT(user)` — signs `{ email, id, username, exp: now+60days }` with `SECRET = 'secret-key'` (`backend/src/config.ts`)
5. Returns `{ user: { email, token, username, bio, image } }`

### Frontend
1. `Login.tsx` calls `login(email, password)` from `conduit.ts`
2. On success: `loadUserIntoApp(user)` — stores token in `localStorage`, sets `axios.defaults.headers.Authorization = 'Token <jwt>'`, dispatches `loadUser` to Redux store
3. On app load (cold start): `App.tsx` reads `localStorage.getItem('token')`, sets axios header, calls `getUser()` to restore session

### Protected Routes
- `AuthMiddleware` (`backend/src/user/auth.middleware.ts`): splits `Authorization: Token <jwt>`, verifies with `jwt.verify(token, SECRET)`, loads user via `UserService.findById(decoded.id)`, sets `req.user`
- Optional auth routes use `@User()` decorator which also attempts JWT decode without throwing

### Seed credentials
- Email: `jcosten0@purevolume.com` / Password: `password`
- Also: `bbebbell1@earthlink.net` / `password`, `zgorey2@livejournal.com` / `password`

---

## Conventions and Tribal Knowledge

### Naming
- Files: `<domain>.<type>.ts` pattern — `article.entity.ts`, `article.service.ts`, `article.controller.ts`, `article.module.ts`
- DTOs in `dto/` subfolder; `index.ts` re-exports all DTOs in the folder
- Entity field names use camelCase in TypeScript, explicit `fieldName` in snake_case for DB columns: `fieldName: 'author_id'`
- Interfaces for response objects named `I<Entity>RO` (Read Object): `IArticleRO`, `IUserRO`

### DTO Pattern
- DTOs are plain classes with `readonly` fields
- Validation decorators (`@IsNotEmpty`, `@IsEmail`) are used selectively — CreateUserDto and LoginUserDto are validated; CreateArticleDto and UpdateUserDto are not
- `ValidationPipe` in `shared/pipes/validation.pipe.ts` is applied manually via `@UsePipes(new ValidationPipe())` on specific controller methods — NOT globally registered

### Validation Approach
Two validation mechanisms coexist:
1. **class-validator on DTOs** (via `ValidationPipe`) — used on `POST /users` and `POST /users/login`
2. **class-validator on Entity** (via `validate(user)` in UserService.create) — validates the entity instance directly

### Error Handling
- HTTP errors thrown as `throw new HttpException({ errors }, statusCode)` or `throw new HttpException('message', HttpStatus.XXX)`
- Frontend `GenericErrors = Record<string, string[]>` — backend must match this shape for frontend error display
- Frontend API calls returning `Result<T, GenericErrors>` use `.match({ ok, err })` pattern

### Request Envelope Pattern
Requests and responses are wrapped in named envelopes per RealWorld API spec:
- Request body: `{ "article": { ... } }` → `@Body('article') articleData`
- Request body: `{ "user": { ... } }` → `@Body('user') userData`
- Response: `{ article: ... }` or `{ articles: [...], articlesCount: N }`

### MikroORM Unit of Work
- `MikroOrmMiddleware` creates a new forked EntityManager per request — this is set up in `AppModule` to run before any other middleware
- Services receive `EntityManager` via constructor injection
- Changes to tracked entities are flushed with `await this.em.flush()` — no explicit `persist()` needed for entities already tracked
- For new untracked entities: `await this.em.persistAndFlush(entity)` (used in addComment)
- Collections must be explicitly populated: `populate: ['followers', 'favorites']` passed to find calls

### `toJSON` Override
Both `User.toJSON(user?)` and `Article.toJSON(user?)` override the default MikroORM serialization to:
- Add computed fields (`favorited`, `following`)
- Replace relation objects with their own `toJSON()` output
- `User.toJSON()` sets a fallback image URL if image is empty

### Config Files
- `backend/mikro-orm.config.ts` — ORM config: host `db` (Docker service name), port 3306, user/password/dbName all `conduit`, entities explicitly listed, `loadStrategy: JOINED`, `debug: true` (logs SQL), `registerRequestContext: false` (NestJS handles it)
- `backend/nestconfig.json` — `{ "language": "ts", "entryFile": "src/main.ts" }` (for NestJS CLI)
- `backend/nest-cli.json` — `{ "collection": "@nestjs/schematics", "sourceRoot": "src" }`

### Dev Servers and Ports
- Backend: `http://localhost:3000` — serves all API at `/api/...`
- Frontend: `http://localhost:3001` — Vite dev server with HMR
- Vite proxy: `vite.config.ts` proxies `/api` → `http://localhost:3000/` (so frontend calls `/api/articles` and it hits the backend)
- Swagger UI: `http://localhost:3000/docs`
- DB: MySQL at port 3306 (host `db` from inside Docker, `localhost` from host)

### Frontend Routing
- Uses `HashRouter` — all routes are hash-based (`/#/`, `/#/login`, `/#/editor/:slug`, etc.)
- Route guards: `createGuestOnlyRoute` redirects logged-in users away from login/register; `createUserOnlyRoute` redirects guests away from settings/editor

### Frontend State Management Pattern
Each feature slice (`*.slice.ts`) follows:
- `initialState` typed interface
- `createSlice` with reducers as the only state mutations
- Async work happens in plain `async` functions (not thunks) that dispatch actions directly via `store.dispatch()`
- `useStoreWithInitializer(getter, initializer)` — subscribes to store + calls `initializer()` on mount
- No middleware (no redux-thunk, no redux-saga) — async is handled imperatively

### Frontend Response Decoding
All API responses are decoded at runtime using `decoders`:
```typescript
object({ article: articleDecoder }).verify(data).article
```
If the shape doesn't match the decoder, this throws at runtime. This catches backend API shape mismatches early.

### Seed Runs on `npm run seed`, NOT Automatically
Despite the comment in the VSCode task, the seeder does NOT run on every backend start. The backend starts and auto-migrates (in `onModuleInit`), but seeding requires the explicit `npm run seed` command. The VSCode task "Run Seed" runs after "Start Backend" is detected as ready.

---

## How to Run / Build / Migrate

### Start Everything (Codespaces / Docker Dev Container)
The `.devcontainer` setup auto-starts everything on `folderOpen` via VSCode tasks:
1. Install root dependencies
2. Install frontend + backend dependencies  
3. Start backend (`ts-node-dev` on port 3000) — also auto-migrates on startup
4. Wait for backend on :3000, then start frontend (Vite on port 3001)
5. Run seed (waits for :3000, then `npm run seed`)

### Manual Commands
```bash
# From repo root
npm run start          # Runs start:backend and start:frontend in parallel

# Backend only
cd backend
npm run start          # ts-node-dev --respawn src/main.ts (auto-migrates on init)
npm run seed           # mikro-orm seeder:run (DatabaseSeeder)

# Frontend only
cd frontend
npm run start          # vite (port 3001)
npm run build          # vite build

# From root
npm run lint           # eslint both
npm run format         # prettier both
npm run submit         # tsx submit.ts <asr-token>
```

### Migrations
Migrations run automatically in `AppModule.onModuleInit()`. To add a migration manually:
1. Create `backend/src/migrations/YourMigration.ts` extending `Migration` with an `up()` method
2. Add to `migrationsList` in `backend/mikro-orm.config.ts`
3. MikroORM tracks which migrations have run in the `mikro_orm_migrations` table

---

## Submit Flow (`submit.ts`)

`npm run submit` runs `tsx submit.ts <asr-token>` where `<asr-token>` is provided by the assessment platform.

What it does:
1. Prompts for name and email (defaults from git config)
2. Creates a git commit with all changes (`git add --all && git commit`)
3. Generates a diff: `git diff origin/rwa/design-and-implementation-v2...HEAD` (excluding plan.md, *.patch, lockfiles, tsconfig)
4. Writes diff to `submission/submission.patch`
5. Checks for warnings:
   - Patch < 100 bytes → likely incomplete
   - No image files in `submission/` → no screenshots
   - No Cline history in VSCode globalStorage → AI usage not demonstrated
   - `plan.md` < 100 bytes → plan missing
6. Creates a ZIP containing: all files in `submission/`, `plan.md`, and Cline chat history (`api_conversation_history.json` + `ui_messages.json` from all Cline tasks)
7. Uploads ZIP to assessment API, prints `submissionId`

### What goes in `submission/`
- **Screenshots** (`.jpg`, `.jpeg`, `.png`, `.gif`, `.bmp`) of acceptance tests passing — **required**, or score is 0
- The patch file is auto-generated; do not manually put code there
- The folder is gitignored (only `.gitignore` itself is tracked)

---

## EXTENSION POINTS

These are the seams where a new feature (new entity, endpoint, or frontend page) would plug in. Described generically — not tied to any specific user story.

### Adding a New Entity

1. **Create entity** in the relevant module folder: `backend/src/<module>/<entity>.entity.ts`
   - Add `@Entity()`, `@PrimaryKey`, `@Property`, and relation decorators
   - Add a `toJSON()` method if the entity needs computed/hidden fields
   - Optionally create a custom repository class extending `EntityRepository<YourEntity>` and reference it via `@Entity({ customRepository: () => YourRepository })`

2. **Register entity in ORM config**: add it to the `entities: [...]` array in `backend/mikro-orm.config.ts`

3. **Create a migration**: add `backend/src/migrations/YourMigration.ts` with the DDL in `up()`, and add it to `migrationsList` in `mikro-orm.config.ts`

4. **Register entity in module**: add to `MikroOrmModule.forFeature({ entities: [..., YourEntity] })` in the feature module's `imports`

### Adding a New Endpoint

1. **Create DTOs** in `backend/src/<module>/dto/`: plain class with `readonly` fields; add class-validator decorators if validation is needed
2. **Add method to service**: inject `EntityRepository<YourEntity>` via `@InjectRepository(YourEntity)` or use `EntityManager`
3. **Add route handler to controller**: use `@Get`, `@Post`, `@Put`, `@Delete` with `@Body`, `@Param`, `@Query`, `@User` decorators; add `@ApiOperation` + `@ApiResponse` for Swagger
4. **Register auth**: if the route requires authentication, add it to the module's `configure(consumer: MiddlewareConsumer)` with `AuthMiddleware`

### Adding a New Frontend Page

1. **Add type definitions** in `frontend/src/types/` for the new data shape + a `decoders` decoder
2. **Add API function** in `frontend/src/services/conduit.ts` — follow the `Result<T, GenericErrors>` pattern for mutating calls, plain async for reads
3. **Create slice** in `frontend/src/components/Pages/<PageName>/<PageName>.slice.ts`: define state interface, `createSlice`, export actions and reducer
4. **Register slice** in `frontend/src/state/store.ts`: import reducer and add to `configureStore({ reducer: { ..., yourSlice } })`
5. **Create page component** in `frontend/src/components/Pages/<PageName>/<PageName>.tsx`: use `useStoreWithInitializer` or `useStore`, dispatch actions, call API functions
6. **Register route** in `frontend/src/components/App/App.tsx`: add a `<Route path="..." element={<YourPage />} />` inside `<Switch>`; wrap in `createUserOnlyRoute` or `createGuestOnlyRoute` if access control is needed

### Adding a Field to an Existing Entity

1. Add `@Property()` to the entity class
2. Add a new migration with the `ALTER TABLE` DDL and add to `migrationsList`
3. Update `toJSON()` if the field needs special handling
4. Update any relevant DTO classes
5. Update frontend type interfaces and decoders in `frontend/src/types/`
6. Update any frontend service calls that send or receive the field

---

## AWS Architecture Notes (from `diagram.png`)

Production deployment: ECS Fargate (1–10 instances, auto-scaled), Aurora MySQL (managed), ALB (load balancer), CloudFront + S3 (static frontend assets).

**What this means for the code**:

- **No in-memory state on the backend** — multiple Fargate instances run simultaneously. Any data stored in module-level variables, singleton caches, or in-process state will be inconsistent across instances. Every request must read from and write to Aurora MySQL.

- **No local disk writes** — Fargate task storage is ephemeral and not shared. File uploads, caches, or any writes to the local filesystem will be lost on container restart or will be invisible to other instances.

- **No single-instance background jobs or timers** — `setInterval`, cron jobs, or any work scheduled in-process will run on every instance independently. Use a database-backed job queue or an external scheduler (e.g., EventBridge) for background work.

- **Sessions must be stateless** — JWT-based auth (already implemented) is correct for this architecture. Avoid server-side sessions or sticky sessions.

- **Database connection limits** — Aurora MySQL has connection limits. ECS auto-scaling multiplies connections by instance count. Avoid creating excess connections; MikroORM's per-request EntityManager pattern is appropriate.

- **The frontend is served from S3/CloudFront** — the React SPA is built (`vite build`) and deployed to S3, not served by the NestJS backend. The Vite proxy (`/api` → `localhost:3000`) is a dev-only convenience; in production, CloudFront routes `/api/` to the ALB/ECS backend.
