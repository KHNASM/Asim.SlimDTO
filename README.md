# Software Requirements Specification (SRS)

**Project Title**: Khan.SlimDTO NuGet Package  
**Version**: 1.0.0  
**Date**: 2024-12-26  
**Authors**: [Your Name]  

## 1. Introduction

### 1.1 Purpose

The purpose of Khan.SlimDTO is to provide a lightweight and efficient library for dynamically generating and managing Data Transfer Objects (DTOs) from entity classes in .NET applications. This library streamlines the process of defining, creating, and managing DTOs through annotations, concurrent caching, and dependency injection, making it ideal for scalable and high-performance applications.

### 1.2 Scope

Khan.SlimDTO will:

- Utilize data annotations to designate DTO properties in entity classes.
- Provide methods for on-the-fly DTO creation by filtering properties based on annotations.
- Allow seamless integration into .NET applications via Dependency Injection (DI).
- Maintain a thread-safe dictionary to store entities and their associated DTOs.
- Offer a generic, extensible architecture to support customization and alternative implementations.
- Ensure race-condition-free operations in asynchronous environments by employing concurrent data structures.

### 1.3 Definitions, Acronyms, and Abbreviations

- **DTO**: Data Transfer Object
- **DI**: Dependency Injection
- **Entity**: A class representing a data model in an application.
- **Concurrent Dictionary**: A thread-safe dictionary for parallel operations.

### 1.4 References

- [.NET Documentation](https://learn.microsoft.com/)
- [NuGet Package Guidelines](https://learn.microsoft.com/en-us/nuget/)

## 2. Functional Requirements

### 2.1 DTO Annotations

- Provide a custom attribute, `DTOPropertyAttribute`, to mark properties in entity classes as part of the DTO.
- Support fine-grained control over which properties should be included in DTOs.

### 2.2 DTO Generation

- Provide a static or service-based method, `CreateDTO<T>()`, to generate DTO objects dynamically.
- Filter entity properties based on the presence of `DTOPropertyAttribute`.

### 2.3 Dependency Injection Support

- Register the service in the DI container using a method like `services.AddSlimDTO()`.
- Allow consumers to inject the service via constructor injection.

### 2.4 Singleton Dictionary

- Implement a thread-safe `ConcurrentDictionary` to:
  - Store mappings between entity types and their generated DTO types.
  - Retrieve or create DTOs as required.

### 2.5 Generic Abstraction

- Utilize generics (`TEntity`, `TDto`) to:
  - Support different types of entities and DTOs.
  - Allow developers to extend and override functionality easily.

### 2.6 Thread Safety

- Employ `ConcurrentDictionary` to avoid race conditions in multi-threaded and asynchronous scenarios.
- Ensure that the library can handle concurrent calls to `CreateDTO` without locking threads.

### 2.7 Performance

- Optimize for performance by:
  - Caching DTOs to reduce repetitive generation overhead.
  - Minimizing reflection-based operations after the initial DTO generation.

### 2.8 Logging and Debugging


- Provide logging hooks for developers to monitor DTO creation and cache hits/misses.
- Allow configurable verbosity through integration with popular logging frameworks (e.g., Serilog).

### 2.9 Sub-Entity DTO Mapping (with EF Navigation Properties)

- The system should handle navigation properties in entities and create corresponding DTOs for these sub-entities.
- Ensure that child entities are included in the DTO by replacing the navigation property with its corresponding DTO.

### 2.10 Skipping Child Entities (Selective DTO Creation)

- Provide functionality to skip the creation of DTOs for specific child entities or navigation properties.
- This can be controlled via flags or method parameters (e.g., `skipChildren: true`).

### 2.11 Lazy Loading of Sub-Entities

- Support lazy loading of child DTOs only when explicitly requested, preventing unnecessary DTO creation.

### 2.12 Nested Entity DTO Creation (Deep DTO Mapping)

- The system should support deep entity graphs and recursively generate DTOs for nested sub-entities.

### 2.13 Handling Collections of Sub-Entities

- Ensure correct handling of collections of related entities by generating DTOs for all elements in the collection.

### 2.14 Excluding Specific Properties or Sub-Entities via Annotations

- Allow developers to annotate properties or sub-entities for exclusion from DTOs using the `DTOPropertyAttribute`.

### 2.15 Circular References

- Implement mechanisms to detect and prevent circular references when generating DTOs for entities with circular relationships.

### 2.16 Customizing DTO Creation Behavior

- Provide extension points or event hooks that allow developers to customize the default DTO creation logic.

### 2.17 Conditional DTO Creation Based on Entity State

- Support conditional DTO creation based on the entity's state (e.g., new, updated, archived).

### 2.18 Handling Complex Properties (e.g., Calculated Fields)

- Support the transformation or calculation of complex properties before they are included in the DTO.

### 2.19 DropChildEntities Method

- **Purpose**: This method allows developers to drop all child entities from the DTO by calling `DropChildren()` on the DTO creation result.
- **Description**: When called on a generated DTO, `DropChildren()` should remove all child entities (i.e., navigation properties or related sub-entities) from the DTO, effectively skipping the transformation of child entities.
- **Example**:
  ```csharp
  var orderDto = CreateDTO<Order>().DropChildren();


### 2.20 Add Custom Property to DTO

- **Purpose**: Allow developers to add custom properties to the DTO after its creation, based on data from the original entity.
- **Description**: The `AddProperty` method enables developers to dynamically add properties to a DTO. This can be used to calculate values, aggregate data, or add business-specific logic that doesn’t exist in the entity but is relevant to the DTO.
- **Example**:
  ```csharp
  public async Task<List<OrderDto>> GetOrdersAsync()
  {
      var orders = await _context.Orders.ToListAsync();
      var orderDtos = orders.Select(o => o.CreateDTO()   // Create DTO from Order
                             .SkipChildren()           // Skip child entities like Products
                             .AddProperty("TotalAmount", o => o.Products.Sum(p => p.Price)) // Add TotalAmount
                             .ToList(); 
      return orderDtos;
  }

## 3. Non-Functional Requirements

### 3.1 Scalability

- The library should handle a large number of entity-DTO mappings without significant performance degradation.

### 3.2 Compatibility

- Compatible with .NET 6 and above.
- Support for ASP.NET Core applications and standalone .NET projects.

### 3.3 Extensibility

- Allow developers to replace the default `DTOPropertyAttribute` or modify the `ConcurrentDictionary` behavior.
- Provide extension points for custom DTO generation logic.

### 3.4 Usability

- Ensure the API is intuitive and well-documented.
- Provide sample projects and comprehensive usage examples.

### 3.5 Reliability

- Ensure thread safety and robust error handling.
- Write unit and integration tests to verify functionality under various scenarios.

## 4. System Models

### 4.1 Component Diagram

**Components**:

- **DTOService**: Handles entity-to-DTO transformation.
- **ConcurrentDictionary**: Caches mappings between entities and DTOs.
- **DTOPropertyAttribute**: Marks properties for inclusion in DTOs.

### 4.2 Sequence Diagram

1. Developer calls `CreateDTO<TEntity>()`.
2. The service checks the `ConcurrentDictionary` for an existing DTO.
   - If found, returns the DTO object.
   - If not found:
     - Analyzes the entity’s properties based on `DTOPropertyAttribute`.
     - Generates the DTO.
     - Updates the `ConcurrentDictionary`.
     - Returns the DTO object.

## 5. Technical Implementation Details

### 5.1 Class Structure

- **DTOPropertyAttribute**: Defines the custom attribute for marking DTO properties.
- **DTOService**: Provides methods to create and retrieve DTOs.
- **SlimDTOExtensions**: Registers the service in the DI container.
- **DTOCache**: Implements the `ConcurrentDictionary` for caching.

### 5.2 Package Dependencies

- .NET Standard Libraries
- Optional: Logging Framework (e.g., Serilog)

## 6. Testing

### 6.1 Unit Tests

- Verify DTO generation with various combinations of annotated and non-annotated properties.
- Ensure thread safety under concurrent operations.

### 6.2 Integration Tests

- Validate DI integration in a sample .NET Core application.

### 6.3 Performance Tests

- Measure latency for DTO generation and cache retrieval.

## 7. Documentation and Support

- **API Documentation**: Auto-generate using XML comments and tools like Swagger.
- **Usage Examples**: Provide examples for common scenarios, including ASP.NET Core integration.
- **Troubleshooting Guide**: Include common issues and solutions.

## 8. Conclusion

Khan.SlimDTO aims to simplify DTO management in .NET applications by offering a performant, extensible, and developer-friendly solution. Its focus on annotations, generics, and thread safety makes it a robust tool for modern development needs.
