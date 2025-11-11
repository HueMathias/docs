# Documentation Entity Framework Core

## 📋 Table des matières

1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Concepts fondamentaux](#concepts-fondamentaux)
4. [Configuration du DbContext](#configuration-du-dbcontext)
5. [Modèles et relations](#modèles-et-relations)
6. [Migrations](#migrations)
7. [Requêtes LINQ](#requêtes-linq)
8. [Opérations CRUD](#opérations-crud)
9. [Patterns avancés](#patterns-avancés)
10. [Bonnes pratiques](#bonnes-pratiques)
11. [Ressources utiles](#ressources-utiles)

---

## Introduction

Entity Framework Core (EF Core) est un ORM (Object-Relational Mapper) moderne pour .NET qui permet de :
- Travailler avec des bases de données en utilisant des objets .NET
- Éviter l'écriture manuelle de requêtes SQL
- Gérer les migrations de schéma de base de données
- Supporter plusieurs fournisseurs de bases de données (SQL Server, PostgreSQL, SQLite, MySQL, etc.)

---

## Installation

### Via .NET CLI

```bash
# Installer EF Core pour SQL Server
dotnet add package Microsoft.EntityFrameworkCore.SqlServer

# Installer les outils de migration
dotnet add package Microsoft.EntityFrameworkCore.Design

# Installer EF Core CLI globalement
dotnet tool install --global dotnet-ef
```

### Packages pour d'autres bases de données

```bash
# PostgreSQL
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL

# MySQL / MariaDB
dotnet add package Pomelo.EntityFrameworkCore.MySql

# SQLite
dotnet add package Microsoft.EntityFrameworkCore.Sqlite

# InMemory (pour les tests)
dotnet add package Microsoft.EntityFrameworkCore.InMemory
```

---

## Concepts fondamentaux

### DbContext

Classe principale qui coordonne les fonctionnalités EF Core pour un modèle de données. Représente une session avec la base de données.

### DbSet<T>

Collection d'entités d'un type spécifique. Permet d'effectuer des requêtes et des opérations CRUD.

### Entity (Entité)

Classe C# qui représente une table dans la base de données.

### Migration

Script qui permet de créer ou modifier le schéma de la base de données de manière versionnée.

---

## Configuration du DbContext

### DbContext basique

```csharp
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    public DbSet<Category> Categories { get; set; }

    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
    }
}
```

### Configuration dans Program.cs (ASP.NET Core)

```csharp
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=MyApp;Trusted_Connection=True;"
  }
}

// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")
    )
);

var app = builder.Build();
```

### Configuration pour application console

```csharp
var optionsBuilder = new DbContextOptionsBuilder<ApplicationDbContext>();
optionsBuilder.UseSqlServer("Server=(localdb)\\mssqllocaldb;Database=MyApp;Trusted_Connection=True;");

using var context = new ApplicationDbContext(optionsBuilder.Options);
```

---

## Modèles et relations

### Entité simple

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public DateTime CreatedAt { get; set; }
    
    // Navigation property
    public int CategoryId { get; set; }
    public Category? Category { get; set; }
}
```

### Relations One-to-Many

```csharp
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    
    // Collection navigation property
    public virtual ICollection<Product> Products { get; set; } = [];
}
```

### Relations Many-to-Many

```csharp
public class Students
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public virtual ICollection<StudentsCourses> StudentsCourses { get; set; } = [];
}

public class Courses
{
    public int Id { get; set; }
    public string Title { get; set; } = string.Empty;
    public virtual ICollection<StudentsCourses> StudentsCourses { get; set; } = [];
}

public class StudentsCourses
{
    public int StudentId { get; set; }
    public int Coursed { get; set; }
    public virtual Courses Courses { get; set; } = new();
    public virtual Students Students { get; set; } = new();
}

// EF Core 5+ configure automatiquement la table de jointure
// Ou configuration explicite :
protected override void OnModelCreating(ModelBuilder builder)
{
    builder.Entity<StudentsCourses>().HasKey(x => new { x.StudentId, x.CourseId });

    builder.Entity<Students>()
        .HasMany(x => x.StudentsCourses)
        .WithOne(x => x.Student)
        .HasForeignKey(x => x.StudentId);

    builder.Entity<Courses>()
        .HasMany(x => x.StudentsCourses)
        .WithOne(x => x.Course)
        .HasForeignKey(x => x.CourseId);
}
```

### Relations One-to-One

```csharp
public class User
{
    public int Id { get; set; }
    public string Username { get; set; } = string.Empty;
    public UserProfile? Profile { get; set; }
}

public class UserProfile
{
    public int Id { get; set; }
    public string Bio { get; set; } = string.Empty;
    public int UserId { get; set; }
    public User User { get; set; } = null!;
}

// Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>()
        .HasOne(u => u.Profile)
        .WithOne(p => p.User)
        .HasForeignKey<UserProfile>(p => p.UserId);
}
```

### Data Annotations

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

[Table("Products")]
public class Product
{
    [Key]
    public int Id { get; set; }
    
    [Required]
    [MaxLength(200)]
    public string Name { get; set; } = string.Empty;
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal Price { get; set; }
    
    [Column("creation_date")]
    public DateTime CreatedAt { get; set; }
}
```

---

## Migrations

### Créer une migration

Commande pour créer une nouvelle migration :
```bash
dotnet ef migrations add <nom>
```

### Appliquer les migrations

```bash
# Appliquer à la base de données
dotnet ef database update

# Appliquer jusqu'à une migration spécifique (précédente ou suivante)
dotnet ef database update <nom>
```

### Supprimer une migration

Commande pour supprimer la dernière migration (non appliquée) :
```bash
dotnet ef migrations remove
```

### Lister les migrations

Commande pour voir toutes les migrations :
```bash
dotnet ef migrations list
```

### Générer un script SQL

```bash
# Générer un script SQL pour toutes les migrations
dotnet ef migrations script

# Script pour une migration spécifique entre deux migrations
dotnet ef migrations script <nom_1> <nom_2>
```

### Migration au démarrage de l'application

```csharp
// Program.cs
var app = builder.Build();

// Appliquer automatiquement les migrations
using (var scope = app.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
    context.Database.Migrate();
}

app.Run();
```

---

## Requêtes LINQ

### Requêtes basiques

```csharp
// Récupérer tous les produits
var products = await context.Products.ToListAsync();

// Filtrer
var expensiveProducts = await context.Products
    .Where(p => p.Price > 100)
    .ToListAsync();

// Rechercher un élément
var product = await context.Products
    .FirstOrDefaultAsync(p => p.Id == 1);

// Rechercher par clé primaire
var product = await context.Products.FindAsync(1);

// Compter
var count = await context.Products.CountAsync();

// Vérifier l'existence
var exists = await context.Products.AnyAsync(p => p.Name == "Laptop");
```

### Requêtes avec relations

```csharp
// Eager Loading (charger les relations)
var productsWithCategory = await context.Products
    .Include(p => p.Category)
    .ToListAsync();

// Charger plusieurs niveaux
var products = await context.Products
    .Include(p => p.Category)
    .ThenInclude(c => c.Subcategories)
    .ToListAsync();

// Filtrer les données incluses
var products = await context.Products
    .Include(p => p.Reviews.Where(r => r.Rating >= 4))
    .ToListAsync();
```

### Projections et sélections

```csharp
// Sélectionner des propriétés spécifiques
var productNames = await context.Products
    .Select(p => p.Name)
    .ToListAsync();

// Projection vers un type anonyme
var productInfo = await context.Products
    .Select(p => new
    {
        p.Id,
        p.Name,
        CategoryName = p.Category.Name
    })
    .ToListAsync();

// Projection vers un DTO
public record ProductDto(int Id, string Name, decimal Price);

var productDtos = await context.Products
    .Select(p => new ProductDto(p.Id, p.Name, p.Price))
    .ToListAsync();
```

### Tri et pagination

```csharp
// Trier
var sortedProducts = await context.Products
    .OrderBy(p => p.Price)
    .ToListAsync();

// Trier par plusieurs critères
var products = await context.Products
    .OrderBy(p => p.Category.Name)
    .ThenByDescending(p => p.Price)
    .ToListAsync();

// Pagination
int pageNumber = 1;
int pageSize = 10;

var pagedProducts = await context.Products
    .Skip((pageNumber - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();
```

### Groupement et agrégation

```csharp
// Grouper
var productsByCategory = await context.Products
    .GroupBy(p => p.Category.Name)
    .Select(g => new
    {
        CategoryName = g.Key,
        Count = g.Count(),
        AveragePrice = g.Average(p => p.Price)
    })
    .ToListAsync();

// Agrégations
var maxPrice = await context.Products.MaxAsync(p => p.Price);
var avgPrice = await context.Products.AverageAsync(p => p.Price);
var totalPrice = await context.Products.SumAsync(p => p.Price);
```

---

## Opérations CRUD

### Create (Créer)

```csharp
// Ajouter une seule entité
var product = new Product
{
    Name = "Laptop",
    Price = 999.99m,
    CreatedAt = DateTime.UtcNow
};

context.Products.Add(product);
await context.SaveChangesAsync();

// Ajouter plusieurs entités
var products = new List<Product>
{
    new Product { Name = "Mouse", Price = 29.99m },
    new Product { Name = "Keyboard", Price = 79.99m }
};

context.Products.AddRange(products);
await context.SaveChangesAsync();
```

### Read (Lire)

```csharp
// Lire par ID
var product = await context.Products.FindAsync(1);

// Lire avec condition
var product = await context.Products
    .FirstOrDefaultAsync(p => p.Name == "Laptop");

// Lire tous
var allProducts = await context.Products.ToListAsync();
```

### Update (Mettre à jour)

```csharp
// Méthode 1 : Récupérer puis modifier
var product = await context.Products.FindAsync(1);
if (product != null)
{
    product.Price = 899.99m;
    await context.SaveChangesAsync();
}

// Méthode 2 : Attach et modifier
var product = new Product { Id = 1, Name = "Laptop", Price = 899.99m };
context.Products.Update(product);
await context.SaveChangesAsync();

// Méthode 3 : Modifier des propriétés spécifiques
var product = new Product { Id = 1 };
context.Products.Attach(product);
context.Entry(product).Property(p => p.Price).IsModified = true;
product.Price = 899.99m;
await context.SaveChangesAsync();
```

### Delete (Supprimer)

```csharp
// Méthode 1 : Récupérer puis supprimer
var product = await context.Products.FindAsync(1);
if (product != null)
{
    context.Products.Remove(product);
    await context.SaveChangesAsync();
}

// Méthode 2 : Supprimer sans récupérer
var product = new Product { Id = 1 };
context.Products.Remove(product);
await context.SaveChangesAsync();

// Supprimer plusieurs entités
var productsToDelete = await context.Products
    .Where(p => p.Price < 10)
    .ToListAsync();
context.Products.RemoveRange(productsToDelete);
await context.SaveChangesAsync();
```

---

## Patterns avancés

### Repository Pattern

```csharp
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

public class Repository<T> : IRepository<T> where T : class
{
    private readonly ApplicationDbContext _context;
    private readonly DbSet<T> _dbSet;

    public Repository(ApplicationDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public async Task<T?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }

    public async Task<T> AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
        return entity;
    }

    public async Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var entity = await _dbSet.FindAsync(id);
        if (entity != null)
        {
            _dbSet.Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
}
```

### Unit of Work Pattern

```csharp
public interface IUnitOfWork : IDisposable
{
    IRepository<Product> Products { get; }
    IRepository<Category> Categories { get; }
    Task<int> SaveChangesAsync();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext _context;

    public UnitOfWork(ApplicationDbContext context)
    {
        _context = context;
        Products = new Repository<Product>(_context);
        Categories = new Repository<Category>(_context);
    }

    public IRepository<Product> Products { get; }
    public IRepository<Category> Categories { get; }

    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }

    public void Dispose()
    {
        _context.Dispose();
    }
}
```

### Specification Pattern

```csharp
public interface ISpecification<T>
{
    Expression<Func<T, bool>> Criteria { get; }
    List<Expression<Func<T, object>>> Includes { get; }
}

public class ProductWithCategorySpecification : ISpecification<Product>
{
    public ProductWithCategorySpecification(decimal minPrice)
    {
        Criteria = p => p.Price >= minPrice;
        Includes = new List<Expression<Func<Product, object>>>
        {
            p => p.Category
        };
    }

    public Expression<Func<Product, bool>> Criteria { get; }
    public List<Expression<Func<Product, object>>> Includes { get; }
}

// Utilisation
public static class SpecificationEvaluator
{
    public static IQueryable<T> GetQuery<T>(
        IQueryable<T> inputQuery, 
        ISpecification<T> spec) where T : class
    {
        var query = inputQuery;

        if (spec.Criteria != null)
        {
            query = query.Where(spec.Criteria);
        }

        query = spec.Includes.Aggregate(query, 
            (current, include) => current.Include(include));

        return query;
    }
}
```

### Transactions

```csharp
// Transaction explicite
using var transaction = await context.Database.BeginTransactionAsync();
try
{
    var product = new Product { Name = "New Product", Price = 100 };
    context.Products.Add(product);
    await context.SaveChangesAsync();

    var category = new Category { Name = "New Category" };
    context.Categories.Add(category);
    await context.SaveChangesAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

### Raw SQL Queries

```csharp
// Requête SQL brute
var products = await context.Products
    .FromSqlRaw("SELECT * FROM Products WHERE Price > {0}", 100)
    .ToListAsync();

// Requête SQL interpolée (plus sûre)
var minPrice = 100m;
var products = await context.Products
    .FromSqlInterpolated($"SELECT * FROM Products WHERE Price > {minPrice}")
    .ToListAsync();

// Exécuter une commande SQL
await context.Database.ExecuteSqlRawAsync(
    "UPDATE Products SET Price = Price * 1.1 WHERE CategoryId = {0}", 
    categoryId);
```

---

## Bonnes pratiques

### Performance

- **Utiliser AsNoTracking()** pour les requêtes en lecture seule
```csharp
var products = await context.Products
    .AsNoTracking()
    .ToListAsync();
```

- **Éviter le N+1 problem** en utilisant Include()
```csharp
// ❌ Mauvais : N+1 queries
var products = await context.Products.ToListAsync();
foreach (var product in products)
{
    var category = product.Category.Name; // Requête pour chaque produit
}

// ✅ Bon : 1 seule requête
var products = await context.Products
    .Include(p => p.Category)
    .ToListAsync();
```

- **Paginer les résultats** pour les grandes collections
- **Projeter** vers des DTOs au lieu de récupérer toutes les colonnes
- **Utiliser des index** sur les colonnes fréquemment recherchées

### Sécurité

- **Toujours utiliser des requêtes paramétrées** (pas de concaténation de chaînes)
- **Valider les données** avant de les sauvegarder
- **Ne jamais exposer** le DbContext directement dans les contrôleurs API
- **Utiliser des DTOs** pour les entrées/sorties API

### Architecture

- **Séparer les préoccupations** : utiliser Repository et Unit of Work
- **Injecter DbContext** via Dependency Injection
- **Utiliser des configurations séparées** pour les entités complexes
```csharp
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.HasKey(p => p.Id);
        builder.Property(p => p.Name).IsRequired().HasMaxLength(200);
        // ...
    }
}

// Dans OnModelCreating
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.ApplyConfiguration(new ProductConfiguration());
}
```

### Tests

- **Utiliser InMemory provider** pour les tests unitaires
```csharp
var options = new DbContextOptionsBuilder<ApplicationDbContext>()
    .UseInMemoryDatabase(databaseName: "TestDb")
    .Options;

using var context = new ApplicationDbContext(options);
```

- **Utiliser SQLite** pour des tests plus proches de la production
```csharp
var connection = new SqliteConnection("DataSource=:memory:");
connection.Open();

var options = new DbContextOptionsBuilder<ApplicationDbContext>()
    .UseSqlite(connection)
    .Options;

using var context = new ApplicationDbContext(options);
context.Database.EnsureCreated();
```

### Gestion des erreurs

- **Gérer les exceptions** DbUpdateException, DbUpdateConcurrencyException
- **Implémenter un retry policy** pour les erreurs transitoires
- **Logger les requêtes SQL** en développement

```csharp
// appsettings.Development.json
{
  "Logging": {
    "LogLevel": {
      "Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
  }
}
```

---

## Ressources utiles

### Documentation officielle
- [EF Core Documentation](https://docs.microsoft.com/ef/core/)
- [EF Core GitHub Repository](https://github.com/dotnet/efcore)
- [Entity Framework Core Tutorial](https://www.entityframeworktutorial.net/efcore/entity-framework-core.aspx)

### Outils
- [EF Core Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EFCorePowerTools) - Extension Visual Studio
- [LINQPad](https://www.linqpad.net/) - Tester des requêtes LINQ
- [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/) - Gérer SQL Server

### Packages utiles
- **EF Core Plus** : Extensions pour EF Core
- **Z.EntityFramework.Plus.EFCore** : Batch operations, audit, etc.
- **AutoMapper** : Mapper entre entités et DTOs

### Commandes CLI essentielles

```bash
# Créer une migration
dotnet ef migrations add NomMigration

# Appliquer les migrations
dotnet ef database update

# Supprimer la dernière migration
dotnet ef migrations remove

# Générer un script SQL
dotnet ef migrations script

# Supprimer la base de données
dotnet ef database drop

# Afficher les informations du contexte
dotnet ef dbcontext info

# Générer des entités depuis une base existante (Scaffold)
dotnet ef dbcontext scaffold "ConnectionString" Microsoft.EntityFrameworkCore.SqlServer -o Models
```
3. Générer et appliquer des migrations
4. Implémenter des opérations CRUD
5. Explorer les patterns Repository et Unit of Work
