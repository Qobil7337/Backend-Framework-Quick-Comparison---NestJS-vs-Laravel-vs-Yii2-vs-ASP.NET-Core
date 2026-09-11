# Backend Framework Quick Comparison
## NestJS vs Laravel vs Yii2 vs ASP.NET Core

A practical code-level comparison for a full-stack developer moving between TypeScript/Node.js, PHP, and .NET.

The examples use the same basic domain:

```text
User
 ├── id
 ├── name
 ├── email
 └── password
```

The goal is not to show the "best" architecture for each framework, but to understand how the **same backend concepts are implemented differently**.

---

# Table of Contents

- [1. Project Structure](#1-project-structure)
- [2. Module / Feature Organization](#2-module--feature-organization)
- [3. Controller](#3-controller)
- [4. Routes](#4-routes)
- [5. Service / Business Logic](#5-service--business-logic)
- [6. Dependency Injection](#6-dependency-injection)
- [7. DTO / Request Validation](#7-dto--request-validation)
- [8. Entity / Model](#8-entity--model)
- [9. Repository / Database Access](#9-repository--database-access)
- [10. CRUD](#10-crud)
- [11. Middleware](#11-middleware)
- [12. Guards / Authorization](#12-guards--authorization)
- [13. Authentication](#13-authentication)
- [14. Pipes / Validation](#14-pipes--validation)
- [15. Interceptors / Filters](#15-interceptors--filters)
- [16. Exception Handling](#16-exception-handling)
- [17. Database Migrations](#17-database-migrations)
- [18. Database Transactions](#18-database-transactions)
- [19. Events](#19-events)
- [20. Queues](#20-queues)
- [21. Scheduled Jobs / Cron](#21-scheduled-jobs--cron)
- [22. Caching](#22-caching)
- [23. WebSockets](#23-websockets)
- [24. Configuration / Environment Variables](#24-configuration--environment-variables)
- [25. Testing](#25-testing)
- [26. Complete Request Lifecycle](#26-complete-request-lifecycle)
- [27. Concept Translation Cheat Sheet](#27-concept-translation-cheat-sheet)

---

# 1. Project Structure

## NestJS

```text
src/
├── app.module.ts
│
├── users/
│   ├── users.module.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   ├── dto/
│   │   ├── create-user.dto.ts
│   │   └── update-user.dto.ts
│   └── entities/
│       └── user.entity.ts
│
└── auth/
    ├── auth.module.ts
    ├── auth.controller.ts
    ├── auth.service.ts
    ├── guards/
    └── strategies/
```

NestJS is strongly organized around **modules + dependency injection**.

---

## Laravel

```text
app/
├── Http/
│   ├── Controllers/
│   │   └── UserController.php
│   ├── Middleware/
│   └── Requests/
│       └── CreateUserRequest.php
│
├── Models/
│   └── User.php
│
└── Services/
    └── UserService.php

routes/
├── api.php
└── web.php

database/
├── migrations/
└── seeders/
```

Laravel does not require a Nest-style module for every feature.

---

## Yii2

```text
controllers/
├── UserController.php

models/
├── User.php
└── CreateUserForm.php

services/
└── UserService.php

modules/
└── admin/

config/
web/
```

Yii2 follows a more traditional MVC structure.

---

## ASP.NET Core

```text
Controllers/
└── UsersController.cs

Services/
├── IUserService.cs
└── UserService.cs

Models/
└── User.cs

DTOs/
└── CreateUserDto.cs

Data/
└── AppDbContext.cs

Middleware/

Program.cs
```

Larger applications often use separate projects:

```text
MyApp.Api
MyApp.Application
MyApp.Domain
MyApp.Infrastructure
```

---

# 2. Module / Feature Organization

## NestJS

Modules are a first-class concept.

```ts
@Module({
  imports: [],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

Then:

```ts
@Module({
  imports: [UsersModule],
})
export class AppModule {}
```

Concept:

```text
UsersModule
 ├── UsersController
 ├── UsersService
 └── UserRepository
```

---

## Laravel

There is no direct equivalent required by the framework.

You normally organize:

```text
UserController
UserService
User model
CreateUserRequest
```

You can create your own feature/module architecture, but Laravel does not force it.

---

## Yii2

Yii2 has actual modules:

```php
class AdminModule extends \yii\base\Module
{
    public $controllerNamespace =
        'app\modules\admin\controllers';
}
```

Structure:

```text
modules/
└── admin/
    ├── controllers/
    ├── models/
    └── views/
```

---

## ASP.NET Core

There is no direct Nest-style `Module`.

Common organization:

```text
Users/
├── UsersController.cs
├── UserService.cs
├── User.cs
└── CreateUserDto.cs
```

Or separate projects:

```text
Api
Application
Domain
Infrastructure
```

---

# 3. Controller

## NestJS

```ts
@Controller('users')
export class UsersController {

  constructor(
    private readonly usersService: UsersService,
  ) {}

  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: number) {
    return this.usersService.findOne(id);
  }

  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}
```

---

## Laravel

```php
class UserController extends Controller
{
    public function __construct(
        private UserService $userService
    ) {}

    public function index()
    {
        return $this->userService->findAll();
    }

    public function show(int $id)
    {
        return $this->userService->findOne($id);
    }

    public function store(CreateUserRequest $request)
    {
        return $this->userService->create(
            $request->validated()
        );
    }
}
```

---

## Yii2

```php
class UserController extends Controller
{
    private UserService $userService;

    public function actionIndex()
    {
        return $this->userService->findAll();
    }

    public function actionView($id)
    {
        return $this->userService->findOne($id);
    }

    public function actionCreate()
    {
        // create user
    }
}
```

Yii2 traditionally uses:

```text
actionIndex()
actionView()
actionCreate()
```

---

## ASP.NET Core

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;

    public UsersController(IUserService userService)
    {
        _userService = userService;
    }

    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        return Ok(await _userService.GetAll());
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> Get(int id)
    {
        return Ok(await _userService.Get(id));
    }

    [HttpPost]
    public async Task<IActionResult> Create(
        CreateUserDto dto)
    {
        return Ok(await _userService.Create(dto));
    }
}
```

---

# 4. Routes

## NestJS

Routes are defined using decorators:

```ts
@Controller('users')
export class UsersController {

  @Get()
  findAll() {}

  @Get(':id')
  findOne() {}

  @Post()
  create() {}

  @Delete(':id')
  delete() {}
}
```

Result:

```text
GET    /users
GET    /users/:id
POST   /users
DELETE /users/:id
```

---

## Laravel

Routes are normally explicit:

```php
Route::get('/users', [
    UserController::class,
    'index'
]);

Route::get('/users/{id}', [
    UserController::class,
    'show'
]);

Route::post('/users', [
    UserController::class,
    'store'
]);

Route::delete('/users/{id}', [
    UserController::class,
    'destroy'
]);
```

Laravel also supports:

```php
Route::apiResource('users', UserController::class);
```

---

## Yii2

Routes map URLs to controller actions.

Example:

```text
/users
/users/view?id=10
```

Controller:

```php
class UserController extends Controller
{
    public function actionIndex()
    {
        // ...
    }

    public function actionView($id)
    {
        // ...
    }
}
```

Pretty URLs can be configured through URL rules.

---

## ASP.NET Core

Routes are usually attributes:

```csharp
[Route("api/users")]
public class UsersController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() {}

    [HttpGet("{id}")]
    public IActionResult Get(int id) {}

    [HttpPost]
    public IActionResult Create(
        CreateUserDto dto) {}
}
```

Result:

```text
GET  /api/users
GET  /api/users/10
POST /api/users
```

---

# 5. Service / Business Logic

## NestJS

```ts
@Injectable()
export class UsersService {

  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  async findAll() {
    return this.repository.find();
  }

  async findOne(id: number) {
    return this.repository.findOne({
      where: { id },
    });
  }

  async create(dto: CreateUserDto) {
    const user = this.repository.create(dto);

    return this.repository.save(user);
  }
}
```

---

## Laravel

Laravel does not require a service layer.

You can write:

```php
class UserController extends Controller
{
    public function index()
    {
        return User::all();
    }
}
```

But for larger applications:

```php
class UserService
{
    public function findAll()
    {
        return User::all();
    }

    public function findOne(int $id)
    {
        return User::findOrFail($id);
    }

    public function create(array $data)
    {
        return User::create($data);
    }
}
```

Controller:

```php
public function __construct(
    private UserService $userService
) {}
```

---

## Yii2

```php
class UserService
{
    public function findAll()
    {
        return User::find()->all();
    }

    public function findOne(int $id)
    {
        return User::findOne($id);
    }

    public function create(array $data)
    {
        $user = new User();
        $user->load($data, '');

        $user->save();

        return $user;
    }
}
```

---

## ASP.NET Core

Interface:

```csharp
public interface IUserService
{
    Task<List<User>> GetAll();
    Task<User?> Get(int id);
    Task<User> Create(CreateUserDto dto);
}
```

Implementation:

```csharp
public class UserService : IUserService
{
    private readonly AppDbContext _db;

    public UserService(AppDbContext db)
    {
        _db = db;
    }

    public async Task<List<User>> GetAll()
    {
        return await _db.Users.ToListAsync();
    }

    public async Task<User?> Get(int id)
    {
        return await _db.Users
            .FirstOrDefaultAsync(x => x.Id == id);
    }

    public async Task<User> Create(
        CreateUserDto dto)
    {
        var user = new User
        {
            Name = dto.Name,
            Email = dto.Email
        };

        _db.Users.Add(user);

        await _db.SaveChangesAsync();

        return user;
    }
}
```

---

# 6. Dependency Injection

## NestJS

```ts
constructor(
  private readonly usersService: UsersService,
) {}
```

Nest automatically creates `UsersService`.

---

## Laravel

Laravel has a service container.

```php
class UserController extends Controller
{
    public function __construct(
        private UserService $userService
    ) {}
}
```

Laravel resolves the dependency automatically in normal cases.

Explicit binding:

```php
$this->app->bind(
    UserService::class,
    function () {
        return new UserService();
    }
);
```

---

## Yii2

Yii2 has a dependency injection container:

```php
Yii::$container->set(
    UserService::class,
    function () {
        return new UserService();
    }
);
```

Dependencies can then be resolved through the container.

---

## ASP.NET Core

Register:

```csharp
builder.Services.AddScoped<
    IUserService,
    UserService
>();
```

Inject:

```csharp
public UsersController(
    IUserService userService)
{
    _userService = userService;
}
```

Common lifetimes:

```text
AddTransient
AddScoped
AddSingleton
```

Roughly:

```text
Transient → new instance each resolution
Scoped    → one instance per request
Singleton  → one instance for application lifetime
```

---

# 7. DTO / Request Validation

## NestJS

```ts
export class CreateUserDto {

  @IsString()
  @IsNotEmpty()
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}
```

Controller:

```ts
@Post()
create(@Body() dto: CreateUserDto) {
  return this.usersService.create(dto);
}
```

---

## Laravel

Create:

```bash
php artisan make:request CreateUserRequest
```

Then:

```php
class CreateUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'name' => [
                'required',
                'string'
            ],

            'email' => [
                'required',
                'email'
            ],

            'password' => [
                'required',
                'string',
                'min:8'
            ],
        ];
    }
}
```

Controller:

```php
public function store(
    CreateUserRequest $request
) {
    return $this->userService->create(
        $request->validated()
    );
}
```

---

## Yii2

Create a form model:

```php
class CreateUserForm extends Model
{
    public $name;
    public $email;
    public $password;

    public function rules()
    {
        return [
            [['name', 'email', 'password'], 'required'],
            ['email', 'email'],
            ['password', 'string', 'min' => 8],
        ];
    }
}
```

Then:

```php
$model = new CreateUserForm();

$model->load($data, '');

if (!$model->validate()) {
    return $model->errors;
}
```

---

## ASP.NET Core

```csharp
public class CreateUserDto
{
    [Required]
    public string Name { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [MinLength(8)]
    public string Password { get; set; }
}
```

Controller:

```csharp
[HttpPost]
public async Task<IActionResult> Create(
    CreateUserDto dto)
{
    return Ok(
        await _userService.Create(dto)
    );
}
```

With `[ApiController]`, model validation happens automatically.

---

# 8. Entity / Model

## NestJS + TypeORM

```ts
@Entity('users')
export class User {

  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;
}
```

---

## Laravel

Laravel calls this a **Model**.

```php
class User extends Model
{
    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    protected $hidden = [
        'password',
    ];
}
```

---

## Yii2

Yii2 commonly uses ActiveRecord:

```php
class User extends ActiveRecord
{
    public static function tableName()
    {
        return 'users';
    }

    public function rules()
    {
        return [
            [['name', 'email'], 'required'],
            ['email', 'email'],
        ];
    }
}
```

---

## ASP.NET Core + EF Core

```csharp
public class User
{
    public int Id { get; set; }

    public string Name { get; set; }

    public string Email { get; set; }

    public string Password { get; set; }
}
```

DbContext:

```csharp
public class AppDbContext : DbContext
{
    public DbSet<User> Users { get; set; }

    public AppDbContext(
        DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }
}
```

---

# 9. Repository / Database Access

## NestJS + TypeORM

```ts
constructor(
  @InjectRepository(User)
  private readonly repository: Repository<User>,
) {}
```

Queries:

```ts
await repository.find();

await repository.findOne({
  where: { id },
});

await repository.findOne({
  where: { email },
});

await repository.save(user);

await repository.delete(id);
```

---

## Laravel

Usually no repository is required.

Eloquent itself provides database access:

```php
User::all();

User::find($id);

User::findOrFail($id);

User::where(
    'email',
    $email
)->first();

User::create($data);

$user->update($data);

$user->delete();
```

---

## Yii2

ActiveRecord provides database access:

```php
User::find()->all();

User::findOne($id);

User::find()
    ->where(['email' => $email])
    ->one();
```

Create:

```php
$user = new User();

$user->name = 'Kobil';
$user->email = 'test@example.com';

$user->save();
```

---

## ASP.NET Core + EF Core

```csharp
await _db.Users.ToListAsync();

await _db.Users
    .FirstOrDefaultAsync(x => x.Id == id);

await _db.Users
    .FirstOrDefaultAsync(
        x => x.Email == email
    );
```

Create:

```csharp
_db.Users.Add(user);

await _db.SaveChangesAsync();
```

Update:

```csharp
user.Name = "New name";

await _db.SaveChangesAsync();
```

Delete:

```csharp
_db.Users.Remove(user);

await _db.SaveChangesAsync();
```

---

# 10. CRUD

The same CRUD operation looks like this.

## CREATE

### NestJS

```ts
const user = repository.create(dto);

return repository.save(user);
```

### Laravel

```php
return User::create($data);
```

### Yii2

```php
$user = new User();

$user->load($data, '');

$user->save();

return $user;
```

### .NET

```csharp
_db.Users.Add(user);

await _db.SaveChangesAsync();

return user;
```

---

## READ

### NestJS

```ts
repository.find();
```

### Laravel

```php
User::all();
```

### Yii2

```php
User::find()->all();
```

### .NET

```csharp
await _db.Users.ToListAsync();
```

---

## READ ONE

### NestJS

```ts
repository.findOne({
  where: { id }
});
```

### Laravel

```php
User::findOrFail($id);
```

### Yii2

```php
User::findOne($id);
```

### .NET

```csharp
await _db.Users
    .FirstOrDefaultAsync(x => x.Id == id);
```

---

## UPDATE

### NestJS

```ts
await repository.update(
  id,
  dto
);
```

### Laravel

```php
$user->update($data);
```

### Yii2

```php
$user->load($data, '');

$user->save();
```

### .NET

```csharp
user.Name = dto.Name;

await _db.SaveChangesAsync();
```

---

## DELETE

### NestJS

```ts
await repository.delete(id);
```

### Laravel

```php
$user->delete();
```

### Yii2

```php
$user->delete();
```

### .NET

```csharp
_db.Users.Remove(user);

await _db.SaveChangesAsync();
```

---

# 11. Middleware

Middleware is code that executes around the HTTP request.

---

## NestJS

```ts
@Injectable()
export class LoggerMiddleware {

  use(req, res, next) {

    console.log(
      req.method,
      req.url
    );

    next();
  }
}
```

Register:

```ts
export class AppModule
  implements NestModule
{
  configure(consumer: MiddlewareConsumer) {

    consumer
      .apply(LoggerMiddleware)
      .forRoutes('*');
  }
}
```

---

## Laravel

```php
class LogRequest
{
    public function handle(
        $request,
        Closure $next
    ) {
        Log::info(
            $request->method()
        );

        return $next($request);
    }
}
```

Apply:

```php
Route::middleware('auth')
    ->get('/profile', ...);
```

---

## Yii2

Yii2 commonly uses filters/behaviors.

Example:

```php
public function behaviors()
{
    return [
        'access' => [
            'class' => AccessControl::class,
            'rules' => [
                [
                    'allow' => true,
                    'roles' => ['@'],
                ],
            ],
        ],
    ];
}
```

---

## ASP.NET Core

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;

    public RequestLoggingMiddleware(
        RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(
        HttpContext context)
    {
        Console.WriteLine(
            context.Request.Path
        );

        await _next(context);
    }
}
```

Register:

```csharp
app.UseMiddleware<
    RequestLoggingMiddleware
>();
```

---

# 12. Guards / Authorization

## NestJS

Authentication guard:

```ts
@Injectable()
export class AuthGuard
  implements CanActivate
{
  canActivate(context: ExecutionContext) {

    const request =
      context.switchToHttp().getRequest();

    return !!request.user;
  }
}
```

Use:

```ts
@UseGuards(AuthGuard)
@Get('profile')
getProfile() {
    ...
}
```

Roles:

```ts
@Roles('admin')
@UseGuards(
  JwtAuthGuard,
  RolesGuard
)
@Delete(':id')
deleteUser() {}
```

---

## Laravel

Authentication:

```php
Route::middleware('auth')
    ->get('/profile', ...);
```

Authorization uses Policies.

```php
class UserPolicy
{
    public function delete(
        User $user,
        User $target
    ) {
        return $user->is_admin;
    }
}
```

Controller:

```php
$this->authorize(
    'delete',
    $target
);
```

---

## Yii2

AccessControl:

```php
public function behaviors()
{
    return [
        'access' => [
            'class' => AccessControl::class,

            'rules' => [
                [
                    'allow' => true,
                    'roles' => ['@'],
                ],
            ],
        ],
    ];
}
```

RBAC can handle roles and permissions.

---

## ASP.NET Core

Authentication:

```csharp
[Authorize]
[HttpGet("profile")]
public IActionResult Profile()
{
    ...
}
```

Role:

```csharp
[Authorize(Roles = "Admin")]
public IActionResult Delete()
{
    ...
}
```

Policy:

```csharp
[Authorize(
    Policy = "CanManageUsers"
)]
public IActionResult Manage()
{
    ...
}
```

---

# 13. Authentication

A typical login flow:

```text
POST /auth/login
       ↓
validate email/password
       ↓
generate token
       ↓
return token
       ↓
client sends token
       ↓
authentication middleware/guard
       ↓
controller
```

---

## NestJS

```ts
@Injectable()
export class AuthService {

  async login(user: User) {

    const payload = {
      sub: user.id,
      email: user.email,
    };

    return {
      access_token:
        this.jwtService.sign(payload),
    };
  }
}
```

JWT guard:

```ts
@UseGuards(JwtAuthGuard)
@Get('profile')
getProfile(@Request() req) {
    return req.user;
}
```

---

## Laravel

With Sanctum:

```php
public function login(
    LoginRequest $request
) {
    $user = User::where(
        'email',
        $request->email
    )->firstOrFail();

    // validate password

    $token = $user
        ->createToken('api')
        ->plainTextToken;

    return [
        'token' => $token,
    ];
}
```

Protected route:

```php
Route::middleware('auth:sanctum')
    ->get('/profile', ...);
```

---

## Yii2

Bearer authentication can be configured:

```php
'authenticator' => [
    'class' => CompositeAuth::class,

    'authMethods' => [
        HttpBearerAuth::class,
    ],
],
```

Identity:

```php
public static function findIdentityByAccessToken(
    $token,
    $type = null
) {
    return static::findOne([
        'access_token' => $token
    ]);
}
```

---

## ASP.NET Core

Configure JWT:

```csharp
builder.Services
    .AddAuthentication(
        JwtBearerDefaults
            .AuthenticationScheme
    )
    .AddJwtBearer(options =>
    {
        // JWT configuration
    });
```

Protected endpoint:

```csharp
[Authorize]
[HttpGet("profile")]
public IActionResult Profile()
{
    return Ok(User);
}
```

---

# 14. Pipes / Validation

## NestJS

```ts
@Get(':id')
findOne(
  @Param('id', ParseIntPipe)
  id: number
) {}
```

DTO validation:

```ts
export class CreateUserDto {

  @IsEmail()
  email: string;

  @MinLength(8)
  password: string;
}
```

Global:

```ts
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    transform: true,
  })
);
```

---

## Laravel

```php
$request->validate([
    'email' => [
        'required',
        'email'
    ],

    'password' => [
        'required',
        'min:8'
    ],
]);
```

Or FormRequest.

---

## Yii2

```php
public function rules()
{
    return [
        ['email', 'required'],
        ['email', 'email'],
        ['password', 'string', 'min' => 8],
    ];
}
```

---

## ASP.NET Core

```csharp
public class CreateUserDto
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [MinLength(8)]
    public string Password { get; set; }
}
```

---

# 15. Interceptors / Filters

## NestJS

```ts
@Injectable()
export class LoggingInterceptor
  implements NestInterceptor
{
  intercept(context, next) {

    const start = Date.now();

    return next.handle().pipe(
      tap(() => {
        console.log(
          Date.now() - start
        );
      })
    );
  }
}
```

Use:

```ts
@UseInterceptors(
  LoggingInterceptor
)
@Get()
findAll() {}
```

---

## Laravel

There isn't a direct `Interceptor` class.

Middleware often covers similar functionality:

```php
public function handle(
    $request,
    Closure $next
) {
    $start = microtime(true);

    $response = $next($request);

    Log::info(
        microtime(true) - $start
    );

    return $response;
}
```

---

## Yii2

Filters can execute before/after controller actions:

```php
public function behaviors()
{
    return [
        'access' => [
            'class' => AccessControl::class,
        ],
    ];
}
```

Yii2 also has behaviors for cross-cutting functionality.

---

## ASP.NET Core

Action filter:

```csharp
public class LoggingFilter
    : IActionFilter
{
    public void OnActionExecuting(
        ActionExecutingContext context)
    {
        Console.WriteLine("Before");
    }

    public void OnActionExecuted(
        ActionExecutedContext context)
    {
        Console.WriteLine("After");
    }
}
```

---

# 16. Exception Handling

## NestJS

```ts
throw new NotFoundException(
    'User not found'
);
```

Custom filter:

```ts
@Catch()
export class AllExceptionsFilter
  implements ExceptionFilter
{
  catch(exception, host) {
    // format/log exception
  }
}
```

---

## Laravel

```php
abort(
    404,
    'User not found'
);
```

Or:

```php
throw new ModelNotFoundException();
```

Laravel's exception handler processes it.

---

## Yii2

```php
throw new NotFoundHttpException(
    'User not found'
);
```

---

## ASP.NET Core

Simple:

```csharp
return NotFound(
    "User not found"
);
```

Or centralized handling:

```csharp
app.UseExceptionHandler("/error");
```

For APIs, custom exception middleware is also common.

---

# 17. Database Migrations

## NestJS / TypeORM

Generate:

```bash
npm run typeorm migration:generate
```

Run:

```bash
npm run typeorm migration:run
```

Migration:

```ts
export class CreateUsers
  implements MigrationInterface
{
  async up(queryRunner: QueryRunner) {

    await queryRunner.createTable(
      new Table({
        name: 'users',
        columns: [
          {
            name: 'id',
            type: 'int',
            isPrimary: true,
            isGenerated: true,
          },
          {
            name: 'email',
            type: 'varchar',
          },
        ],
      })
    );
  }

  async down(queryRunner: QueryRunner) {
    await queryRunner.dropTable('users');
  }
}
```

---

## Laravel

Create:

```bash
php artisan make:migration create_users_table
```

Migration:

```php
Schema::create('users', function (
    Blueprint $table
) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->timestamps();
});
```

Run:

```bash
php artisan migrate
```

Rollback:

```bash
php artisan migrate:rollback
```

---

## Yii2

Create:

```bash
php yii migrate/create create_users_table
```

Migration:

```php
$this->createTable('users', [
    'id' => $this->primaryKey(),
    'name' => $this->string()->notNull(),
    'email' => $this->string()->notNull(),
]);
```

Run:

```bash
php yii migrate
```

Rollback:

```bash
php yii migrate/down
```

---

## ASP.NET Core / EF Core

Create:

```bash
dotnet ef migrations add CreateUsers
```

Apply:

```bash
dotnet ef database update
```

EF generates the migration based on your model.

---

# 18. Database Transactions

## NestJS / TypeORM

```ts
await this.dataSource.transaction(
  async manager => {

    const user = await manager.save(
      User,
      userData
    );

    await manager.save(
      Profile,
      profileData
    );
  }
);
```

---

## Laravel

```php
DB::transaction(function () {

    $user = User::create($data);

    Profile::create([
        'user_id' => $user->id,
    ]);
});
```

---

## Yii2

```php
$transaction =
    Yii::$app->db->beginTransaction();

try {

    $user->save();
    $profile->save();

    $transaction->commit();

} catch (\Throwable $e) {

    $transaction->rollBack();

    throw $e;
}
```

---

## ASP.NET Core

```csharp
using var transaction =
    await _db.Database
        .BeginTransactionAsync();

try
{
    _db.Users.Add(user);

    await _db.SaveChangesAsync();

    _db.Profiles.Add(profile);

    await _db.SaveChangesAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();

    throw;
}
```

---

# 19. Events

## NestJS

Emit:

```ts
this.eventEmitter.emit(
    'user.created',
    user
);
```

Listen:

```ts
@OnEvent('user.created')
handleUserCreated(user: User) {
    console.log(user);
}
```

---

## Laravel

Event:

```php
event(
    new UserCreated($user)
);
```

Listener:

```php
class SendWelcomeEmail
{
    public function handle(
        UserCreated $event
    ) {
        // send email
    }
}
```

---

## Yii2

Yii2 has an event system:

```php
$this->on(
    User::EVENT_AFTER_INSERT,
    function ($event) {
        // ...
    }
);
```

---

## ASP.NET Core

You can implement domain events yourself.

A common ecosystem option is MediatR:

```csharp
await _mediator.Publish(
    new UserCreatedEvent(user.Id)
);
```

Handler:

```csharp
public class UserCreatedHandler
    : INotificationHandler<UserCreatedEvent>
{
    public async Task Handle(
        UserCreatedEvent notification,
        CancellationToken cancellationToken)
    {
        // ...
    }
}
```

---

# 20. Queues

## NestJS

Example with BullMQ:

```ts
await emailQueue.add(
    'welcome-email',
    {
        userId: user.id
    }
);
```

Processor:

```ts
@Processor('email')
export class EmailProcessor {

  @Process('welcome-email')
  async handle(job: Job) {
    // send email
  }
}
```

---

## Laravel

Queues are first-class:

```php
SendWelcomeEmail::dispatch($user);
```

Worker:

```bash
php artisan queue:work
```

---

## Yii2

Usually external infrastructure/extensions:

```text
RabbitMQ
Redis
Kafka
```

For example, the application may publish:

```text
user.created
```

to RabbitMQ and a worker consumes it.

---

## ASP.NET Core

Common options:

```text
BackgroundService
Hangfire
MassTransit
RabbitMQ
Azure Service Bus
Kafka
```

Example BackgroundService:

```csharp
public class EmailWorker
    : BackgroundService
{
    protected override async Task ExecuteAsync(
        CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await ProcessEmails();

            await Task.Delay(
                1000,
                stoppingToken
            );
        }
    }
}
```

---

# 21. Scheduled Jobs / Cron

## NestJS

```ts
@Cron('0 */5 * * * *')
handleCron() {
    // every 5 minutes
}
```

---

## Laravel

```php
Schedule::call(function () {
    // task
})->everyFiveMinutes();
```

---

## Yii2

Create a console command:

```bash
php yii cleanup
```

Then Linux cron:

```text
*/5 * * * * php yii cleanup
```

---

## ASP.NET Core

Use `BackgroundService`, Quartz, Hangfire, etc.

```csharp
public class CleanupWorker
    : BackgroundService
{
    protected override async Task ExecuteAsync(
        CancellationToken token)
    {
        while (!token.IsCancellationRequested)
        {
            await Cleanup();

            await Task.Delay(
                TimeSpan.FromMinutes(5),
                token
            );
        }
    }
}
```

---

# 22. Caching

## NestJS / Redis

```ts
await redis.set(
    `user:${id}`,
    JSON.stringify(user),
    'EX',
    3600
);
```

Get:

```ts
const cached = await redis.get(
    `user:${id}`
);
```

---

## Laravel

```php
$user = Cache::remember(
    "user:$id",
    3600,
    function () use ($id) {
        return User::find($id);
    }
);
```

---

## Yii2

```php
$user = Yii::$app->cache->get(
    "user:$id"
);

if ($user === false) {

    $user = User::findOne($id);

    Yii::$app->cache->set(
        "user:$id",
        $user,
        3600
    );
}
```

---

## ASP.NET Core

Using `IDistributedCache`:

```csharp
var value =
    await cache.GetStringAsync(
        $"user:{id}"
    );
```

Set:

```csharp
await cache.SetStringAsync(
    $"user:{id}",
    json
);
```

Redis can be used as the distributed cache.

---

# 23. WebSockets

## NestJS

```ts
@WebSocketGateway()
export class ChatGateway {

    @SubscribeMessage('message')
    handleMessage(
        client,
        data
    ) {
        client.emit(
            'message',
            data
        );
    }
}
```

---

## Laravel

Laravel commonly uses broadcasting.

```php
broadcast(
    new MessageSent($message)
);
```

The frontend can listen using Laravel Echo or another WebSocket-compatible client.

---

## Yii2

Yii2 normally relies on extensions or external WebSocket infrastructure rather than a core WebSocket gateway abstraction like NestJS.

---

## ASP.NET Core

SignalR:

```csharp
public class ChatHub : Hub
{
    public async Task SendMessage(
        string message)
    {
        await Clients.All.SendAsync(
            "message",
            message
        );
    }
}
```

Client:

```text
Client
  ↕
SignalR
  ↕
ChatHub
```

The closest comparison is:

```text
NestJS Gateway
        ≈
ASP.NET SignalR Hub
```

---

# 24. Configuration / Environment Variables

## NestJS

`.env`:

```env
PORT=3000
DATABASE_URL=postgresql://...
JWT_SECRET=secret
REDIS_HOST=localhost
```

Load:

```ts
ConfigModule.forRoot({
  isGlobal: true,
});
```

Use:

```ts
constructor(
  private config: ConfigService
) {}

const port =
  this.config.get<number>('PORT');
```

---

## Laravel

`.env`:

```env
APP_NAME=MyApp
APP_ENV=local

DB_CONNECTION=pgsql
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=myapp
DB_USERNAME=postgres
DB_PASSWORD=password
```

Use:

```php
env('APP_ENV');
```

Usually configuration is accessed through:

```php
config('app.name');
```

---

## Yii2

Environment variables:

```env
DB_HOST=localhost
DB_NAME=myapp
```

Configuration:

```php
'db' => [
    'class' => Connection::class,
    'dsn' => 'pgsql:host=localhost;dbname=myapp',
    'username' => 'postgres',
    'password' => 'password',
],
```

---

## ASP.NET Core

`appsettings.json`:

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Database=myapp"
  },

  "Jwt": {
    "Secret": "secret"
  }
}
```

Environment variables can override configuration.

Read:

```csharp
var connection =
    builder.Configuration
        .GetConnectionString("Default");
```

---

# 25. Testing

## NestJS

Jest:

```ts
describe('UsersService', () => {

  it('should find user', async () => {

    const result =
      await service.findOne(1);

    expect(result.id).toBe(1);
  });

});
```

E2E:

```ts
request(app.getHttpServer())
    .get('/users')
    .expect(200);
```

---

## Laravel

Pest/PHPUnit:

```php
it('returns users', function () {

    $response = $this->get('/api/users');

    $response->assertStatus(200);
});
```

Or PHPUnit:

```php
public function test_users()
{
    $response = $this->get('/api/users');

    $response->assertStatus(200);
}
```

---

## Yii2

Commonly PHPUnit/Codeception.

```php
public function testFindUser()
{
    $user = User::findOne(1);

    $this->assertNotNull($user);
}
```

---

## ASP.NET Core

Common choices:

```text
xUnit
NUnit
MSTest
```

Example xUnit:

```csharp
[Fact]
public async Task
GetUser_ReturnsUser()
{
    var user =
        await service.Get(1);

    Assert.NotNull(user);
    Assert.Equal(1, user.Id);
}
```

---

# 26. Complete Request Lifecycle

This is one of the most important things to understand.

Suppose the client sends:

```http
POST /users
Authorization: Bearer xxx

{
    "name": "Kobil",
    "email": "kobil@example.com"
}
```

---

## NestJS

```text
HTTP Request
     ↓
Middleware
     ↓
Guards
     ↓
Interceptors
     ↓
Pipes / Validation
     ↓
Controller
     ↓
Service
     ↓
Repository / ORM
     ↓
PostgreSQL
     ↓
Service
     ↓
Controller
     ↓
Interceptor
     ↓
HTTP Response
```

Code:

```ts
@UseGuards(JwtAuthGuard)
@Post()
create(
  @Body() dto: CreateUserDto
) {
  return this.usersService.create(dto);
}
```

---

## Laravel

```text
HTTP Request
     ↓
Middleware
     ↓
Route
     ↓
FormRequest validation
     ↓
Controller
     ↓
Service
     ↓
Eloquent
     ↓
Database
     ↓
Controller
     ↓
Response
```

Code:

```php
Route::middleware('auth:sanctum')
    ->post(
        '/users',
        [UserController::class, 'store']
    );
```

---

## Yii2

```text
HTTP Request
     ↓
Application
     ↓
Filters / Behaviors
     ↓
Controller Action
     ↓
Form Model validation
     ↓
Service / Model
     ↓
ActiveRecord
     ↓
Database
     ↓
Response
```

---

## ASP.NET Core

```text
HTTP Request
     ↓
Middleware pipeline
     ↓
Authentication
     ↓
Authorization
     ↓
Model binding
     ↓
Model validation
     ↓
Controller
     ↓
Service
     ↓
EF Core
     ↓
Database
     ↓
Controller
     ↓
HTTP Response
```

---

# 27. Concept Translation Cheat Sheet

This is the section to check when you forget:

| Concept | NestJS | Laravel | Yii2 | ASP.NET Core |
|---|---|---|---|---|
| Main organization | `Module` | Convention / folders | `Module` | Project / feature |
| Controller | `@Controller()` | `Controller` | `Controller` | `[ApiController]` |
| Route | `@Get()` | `Route::get()` | URL/action | `[HttpGet]` |
| Service | `@Injectable()` | Service class | Service class | Service class |
| DI | Built-in | Service Container | DI Container | Built-in |
| DTO | DTO class | FormRequest / DTO | Form Model | DTO |
| Validation | `class-validator` | FormRequest | `rules()` | DataAnnotations / FluentValidation |
| Entity | TypeORM Entity | Eloquent Model | ActiveRecord | EF Entity |
| ORM | TypeORM / Prisma | Eloquent | ActiveRecord | EF Core |
| Repository | TypeORM Repository | Usually Eloquent | ActiveRecord | DbContext |
| Migration | TypeORM / Prisma | `artisan migrate` | `yii migrate` | EF migrations |
| Middleware | Middleware | Middleware | Filters / Behaviors | Middleware |
| Authentication | Guard + Strategy | Middleware + Sanctum/Passport | Auth components | Authentication middleware |
| Authorization | Guards + Roles | Policies / Gates | RBAC | `[Authorize]` / Policies |
| Pipe | Pipe | Validation | Rules | Model validation |
| Interceptor | Interceptor | Middleware / Events | Filters / Behaviors | Action Filter |
| Exception Filter | Exception Filter | Exception Handler | Exception handling | Exception Middleware |
| Cache | Cache / Redis | Cache | Cache | IMemoryCache / Redis |
| Queue | BullMQ / RabbitMQ/etc. | Queue | Extensions | Hangfire / MassTransit/etc. |
| Event | EventEmitter | Events / Listeners | Events | Domain Events / MediatR |
| Cron | `@Cron()` | Scheduler | Console + cron | BackgroundService |
| WebSocket | Gateway | Broadcasting | Extensions | SignalR Hub |
| GraphQL | Resolver | Lighthouse | Extensions | Hot Chocolate/etc. |
| Tests | Jest | PHPUnit / Pest | PHPUnit / Codeception | xUnit / NUnit |

---

# 28. The Four Frameworks in One Picture

## NestJS

```text
Module
 │
 ├── Controller
 │      ↓
 │    DTO
 │      ↓
 ├── Service
 │      ↓
 ├── Repository
 │      ↓
 └── Entity
        ↓
    Database
```

With cross-cutting layers:

```text
Middleware
    ↓
Guards
    ↓
Interceptors
    ↓
Pipes
    ↓
Controller
    ↓
Service
```

---

## Laravel

```text
Route
  ↓
Middleware
  ↓
FormRequest
  ↓
Controller
  ↓
Service (optional)
  ↓
Eloquent Model
  ↓
Database
```

Authorization:

```text
Middleware
   +
Policy / Gate
```

---

## Yii2

```text
Route
  ↓
Application
  ↓
Filters / Behaviors
  ↓
Controller
  ↓
Form Model / Validation
  ↓
Service
  ↓
ActiveRecord
  ↓
Database
```

---

## ASP.NET Core

```text
Middleware Pipeline
       ↓
Authentication
       ↓
Authorization
       ↓
Model Binding
       ↓
Validation
       ↓
Controller
       ↓
Service
       ↓
DbContext / Repository
       ↓
EF Core
       ↓
Database
```

---

# 29. Most Important Differences

## NestJS vs ASP.NET Core

These are architecturally the most similar.

Both have:

```text
Dependency Injection
Controllers
Services
Guards / Authorization
Middleware
DTOs
ORM
Interceptors / Filters
WebSockets
Background processing
```

The syntax is different, but the mental model is very similar.

```text
NestJS                         ASP.NET Core

@Module()                      Project / DI registration

@Controller()                  [ApiController]

@Get()                         [HttpGet]

@Post()                        [HttpPost]

@Injectable()                  Service class

constructor(...)               constructor(...)

@UseGuards()                   [Authorize]

@UseInterceptors()             IActionFilter

Pipe                            Model validation

TypeORM                         EF Core

Gateway                         SignalR Hub
```

---

# 30. Laravel vs Yii2

These are also conceptually closer to each other.

Both commonly use:

```text
MVC
Active Record
Controllers
Models
Validation rules
Middleware / Filters
Migrations
Events
Caching
```

The biggest practical difference is the ecosystem and conventions.

Laravel:

```php
User::where('email', $email)->first();
```

Yii2:

```php
User::find()
    ->where(['email' => $email])
    ->one();
```

Same idea.

Different syntax.

---

# 31. The Most Important Mental Model

When switching frameworks, don't ask:

> "What is the NestJS equivalent of this file?"

Ask:

> "Which component is responsible for this job in this framework?"

For example:

### "I need to protect an endpoint."

NestJS:

```ts
@UseGuards(JwtAuthGuard)
```

Laravel:

```php
Route::middleware('auth:sanctum')
```

Yii2:

```php
AccessControl
```

.NET:

```csharp
[Authorize]
```

Different syntax.

Same responsibility:

```text
Authenticate / authorize request
```

---

### "I need to validate incoming data."

NestJS:

```ts
class-validator
```

Laravel:

```php
FormRequest
```

Yii2:

```php
rules()
```

.NET:

```csharp
DataAnnotations
```

Same responsibility:

```text
Validate request
```

---

### "I need to query users."

NestJS:

```ts
repository.findOne(...)
```

Laravel:

```php
User::find(...)
```

Yii2:

```php
User::findOne(...)
```

.NET:

```csharp
_db.Users.FirstOrDefaultAsync(...)
```

Same responsibility:

```text
Database access
```

---

# Final Cheat Sheet

If you only remember one thing:

```text
                         RESPONSIBILITY
                              │
       ┌──────────────────────┼──────────────────────┐
       │                      │                      │
    NestJS                 Laravel                 Yii2
       │                      │                      │
       │                      │                      │
 Controller              Controller              Controller
 Service                 Service*                Service*
 DTO                     FormRequest             Form Model
 Entity                  Model                   ActiveRecord
 Repository              Eloquent                ActiveRecord
 Guard                   Middleware              AccessControl
 Pipe                    Validation              rules()
 Interceptor             Middleware              Filter
 Module                  Convention              Module
 Gateway                 Broadcasting            Extension
```

And .NET:

```text
                    ASP.NET Core
                         │
                     Controller
                         ↓
                       DTO
                         ↓
                      Service
                         ↓
                     DbContext
                         ↓
                      EF Core
                         ↓
                      Database
```

The `*` means **optional**. Laravel and Yii2 don't force you to have a Service layer in the same way a typical NestJS architecture does.

The key skill is therefore not memorizing 100 decorators/classes. It is recognizing the **backend responsibility**:

```text
HTTP routing
    ↓
Request validation
    ↓
Authentication
    ↓
Authorization
    ↓
Controller
    ↓
Business logic
    ↓
Data access
    ↓
Database
    ↓
Response
```

Once you understand that pipeline, moving between **NestJS, Laravel, Yii2, and ASP.NET Core** becomes mostly a matter of learning each framework's syntax and conventions.