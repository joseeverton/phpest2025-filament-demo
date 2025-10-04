<p align="center"><a href="https://phppiaui.com.br" target="_blank"><img src="https://eventiza.com.br/_next/image?url=https%3A%2F%2Ffirebasestorage.googleapis.com%2Fv0%2Fb%2Feventiza-pro.appspot.com%2Fo%2Fevents%252Fqu7JYz7HJxHqIvKeusDQ%252Fc64ddd6f-7a36-4a07-b0a4-36bc6fec3071.jpeg%3Falt%3Dmedia%26token%3Da28e6f00-6c20-4bb9-a9b7-0301236bfd3c&w=1920&q=75" width="400" alt="PHPest2025"></a></p>

# Filament Demo – PHPeste 2025

Exemplo de uso do [Filament](https://filamentphp.com/) no Laravel para criação rápida de CRUDs, relacionamentos e permissões.  
Este repositório foi criado para a palestra **"Filament: o atalho para CRUDs poderosos no Laravel"** apresentada por **José Everton da Silva Fontenele** no **PHPeste 2025**.

---

## 🚀 O que você vai ver aqui
- Criar um CRUD em minutos com Filament
- Trabalhar com relacionamentos (Post → Category)
- Aplicar permissões com Spatie Roles
- Customizar formulários (upload de imagem, slug, etc.)

---

## 📦 Instalação do projeto

### 1. Clonar repositório
```bash
git clone https://github.com/joseeverton/phpest2025-filament-demo.git
cd filament-demo
```
### 2. Instalar dependências
```bash
composer install
cp .env.example .env
php artisan key:generate
```
### 3. Configurar banco de dados
Configure o banco de dados no `.env` e rode:
```
php artisan migrate
```
### 4. Instalar Filament
```bash
composer require filament/filament
php artisan filament:install --panels
```
### 4.1 Criar usuário admin:
```bash
php artisan make:filament-user
```
## 📝 Modelos e Migrations
### Category
```bash
php artisan make:model Category -m
```
#### database/migrations/xxxx_create_categories_table.php
```php
Schema::create('categories', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->timestamps();
});
```
### Post
```bash
php artisan make:model Post -m
```
#### database/migrations/xxxx_create_posts_table.php
```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content');
    $table->string('image')->nullable();
    $table->foreignId('category_id')->constrained()->cascadeOnDelete();
    $table->timestamps();
});
```
### Rodar migrations
```bash
php artisan migrate
```
## ⚡ Criando CRUDs com Filament
### 1. Category Resource
```bash
php artisan make:filament-resource Category
```
#### app/Filament/Resources/CategoryResource.php
```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            TextInput::make('name')->required()->maxLength(255),
        ]);
}

public static function table(Table $table): Table
{
    return $table
        ->columns([
            TextColumn::make('id')->sortable(),
            TextColumn::make('name')->searchable()->sortable(),
            TextColumn::make('created_at')->dateTime()->sortable(),
        ])
        ->filters([
            //
        ])
        ->actions([
            Tables\Actions\EditAction::make(),
        ])
        ->bulkActions([
            Tables\Actions\DeleteBulkAction::make(),
        ]);
}
``` 
### 2. Post Resource
```bash
php artisan make:filament-resource Post
```
#### app/Filament/Resources/PostResource.php
```php
use Filament\Forms\Components\Select;
use Filament\Forms\Components\Textarea;
use Filament\Forms\Components\TextInput;
use Filament\Forms\Components\FileUpload;
use Filament\Tables\Columns\ImageColumn;
```
```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            TextInput::make('title')->required()->maxLength(255),
            Textarea::make('content')->required(),
            FileUpload::make('image')->image()->nullable(),
            Select::make('category_id')
                ->relationship('category', 'name')
                ->required(),
        ]);
}
public static function table(Table $table): Table
{
    return $table
        ->columns([
            TextColumn::make('id')->sortable(),
            TextColumn::make('title')->searchable()->sortable(),
            ImageColumn::make('image')->rounded(),
            TextColumn::make('category.name')->label('Category')->sortable()->searchable(),
            TextColumn::make('created_at')->dateTime()->sortable(),
        ])
        ->filters([
            //
        ])
        ->actions([
            Tables\Actions\EditAction::make(),
        ])
        ->bulkActions([
            Tables\Actions\DeleteBulkAction::make(),
        ]);
}
```
## 🔗 Relacionamentos
### app/Models/Post.php
```php
public function category()
{
    return $this->belongsTo(Category::class);
}
```
### app/Models/Category.php
```php
public function posts()
{
    return $this->hasMany(Post::class);
}
```
## 🔐 Permissões com Spatie Roles
### 1. Instalar Spatie Roles
```bash
composer require spatie/laravel-permission
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
php artisan migrate
```
### 2. Configurar User Model
```php
use Spatie\Permission\Traits\HasRoles;
class User extends Authenticatable
{
    use HasRoles;
    // ...
}
```
### 3. Criar Roles e Permissions
```php
php artisan make:seeder RolesAndPermissionsSeeder
```
#### database/seeders/RolesAndPermissionsSeeder.php
```php
use Spatie\Permission\Models\Role;
use Spatie\Permission\Models\Permission;
use Illuminate\Database\Seeder;
class RolesAndPermissionsSeeder extends Seeder
{
    public function run()
    {
        // Reset cached roles and permissions
        app()[\Spatie\Permission\PermissionRegistrar::class]->forgetCachedPermissions();

        // create permissions
        Permission::create(['name' => 'manage categories']);
        Permission::create(['name' => 'manage posts']);

        // create roles and assign existing permissions
        $role = Role::create(['name' => 'admin']);
        $role->givePermissionTo(Permission::all());

        $role = Role::create(['name' => 'editor']);
        $role->givePermissionTo('manage posts');
    }
}
```
### 4. Rodar Seeder
```bash
php artisan db:seed --class=RolesAndPermissionsSeeder
```
### 5. Atribuir Role ao Usuário
```bash
php artisan tinker
$user = \App\Models\User::find(1);
$user->assignRole('admin');
``` 
### 6. Proteger Resources no Filament
#### app/Filament/Resources/CategoryResource.php
```php
public static function canViewAny(): bool
{
    return auth()->user()->can('manage categories');
}
```
#### app/Filament/Resources/PostResource.php
```php
public static function canViewAny(): bool
{
    return auth()->user()->can('manage posts');
}
``` 
## 🎉 Pronto! Agora você tem um sistema básico de CRUD com Filament, relacionamentos e permissões.
Acesse o painel administrativo em `/admin` e comece a gerenciar suas categorias e posts!
## 📚 Recursos adicionais
- [Documentação do Filament](https://filamentphp.com/docs)
- [Documentação do Spatie Roles](https://spatie.be/docs/laravel-permission/v5/introduction)
- [Laravel Documentation](https://laravel.com/docs)

