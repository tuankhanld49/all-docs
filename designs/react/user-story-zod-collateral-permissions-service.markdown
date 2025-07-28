# User Story: Standardize Navigation with Zod-Based Complex Type Validation, Permission Checking, and Navigation Service in Microfrontend Application

## Title
As a developer, I want to standardize navigation and data passing with a dedicated navigation service layer that uses the `zod` library for complex type validation and permission checking for `ICollateralProperty` in the microfrontend application, so that only authorized users can access destination pages with correctly typed data, navigation logic is centralized, and the system is more modular, maintainable, and testable.

## Description
In our React microfrontend application, a shell application hosts multiple miniapps. Navigation is defined in a `webPath` object (e.g., `{'path1': '/path1', 'path2': '/path2'}`) and managed by a `navigate` function that passes `appData` stored in the shell’s context. The `appData` includes complex types like `ICollateralProperty` (`{ id: string, status: string, valuations: array, titles: array, permissions: string[] }`) and primitives like `nabValuationId`. Currently, navigation logic (validation, permission checks) is embedded in the shell, making it hard to maintain and test. There’s also no mechanism to ensure users have permissions to access destination pages, risking unauthorized access. Validating complex types manually is error-prone. We need a `NavigationService` to centralize navigation logic, validate `ICollateralProperty` using `zod`, check permissions via `ICollateralProperty.permissions`, and refactor existing paths efficiently.

## As a
Developer

## I want to
Standardize navigation with a `NavigationService` that handles validation of complex types like `ICollateralProperty` using `zod` schemas, checks user permissions before navigating, uses `nabValuationId` for relevant paths, and centralizes navigation logic

## So that
I can ensure only authorized users access destination pages with correctly typed data, reduce runtime errors, enhance security, improve debugging, centralize navigation logic for maintainability and testability, and refactor existing paths efficiently in our microfrontend architecture.

## Acceptance Criteria

1. **Navigation Service Layer**:
   - A `NavigationService` class encapsulates all navigation logic, including:
     - Looking up path configurations in `pathRegistry`.
     - Validating `appData` using `zod` schemas (e.g., `ICollateralPropertySchema`).
     - Checking permissions via `ICollateralProperty.permissions`.
     - Storing `appData` and triggering navigation with `react-router-dom`.
   - The service is injectable with dependencies (e.g., `navigate`, `setAppData`) for testability.
   - Accessible via `ShellContext` for use by miniapps.

2. **Path Registry with Zod Schema and Permission Support**:
   - A centralized `pathRegistry` maps each path key to its path, required/optional `appData` fields (e.g., `property: ICollateralProperty`, `nabValuationId: string`), and required permissions (e.g., `view-collateral`).
   - Example: For `path1`, required fields are `property: ICollateralProperty` (validated by `zod` with `id`, `status`, `valuations`, `titles`, `permissions`) and `nabValuationId: string`, with permissions `view-collateral` or `edit-collateral`.
   - Defined using TypeScript interfaces and `zod` schemas for type safety.
   - Supports dynamic updates for paths and permissions.

3. **Navigation with Zod Validation and Permission Checking**:
   - The `NavigationService`’s `navigateTo` method accepts a `pathKey` and `appData`.
   - Validates fields using `zod` schemas for complex types and type checks for primitives.
   - Checks if `ICollateralProperty.permissions` includes at least one `requiredPermissions`.
   - Failures (invalid data or permissions) throw errors with `zod` details or permission messages (e.g., “Invalid ICollateralProperty: Expected string for id” or “Insufficient permissions: view-collateral”).
   - Stores `appData` in the shell’s context and navigates to the destination path.
   - TypeScript infers types from `zod` schemas.

4. **Data Access in Destination Pages**:
   - Destination pages retrieve `appData` using the current `pathname`.
   - If data is missing or invalid (e.g., fails `ICollateralPropertySchema.safeParse`), a user-friendly error message with `zod` details is displayed.
   - Pages are wrapped in an `ErrorBoundary` to prevent crashes.

5. **Refactoring Existing Paths**:
   - A script converts `webPath` into `pathRegistry`, including `zod` schemas for `ICollateralProperty`, `nabValuationId`, and default permissions.
   - Minimizes changes to navigation calls (e.g., `navigationService.navigateTo('path1', { required: { property: ICollateralProperty, nabValuationId } })`).
   - Provides a script to refactor many paths efficiently.
   - Validates paths for complete type and permission definitions.

6. **Debugging Support**:
   - `NavigationService` logs path key, destination path, `appData`, validation results, and permission checks (e.g., “Invalid ICollateralProperty” or “Insufficient permissions”).
   - Logs include timestamps and source page for tracing.
   - Debug mode warns about missing optional fields.
   - Destination page errors are logged with `zod` details and context.

7. **Type Safety and Developer Experience**:
   - TypeScript enforces type safety for `pathKey`, `appData`, and permissions, with autocompletion for `ICollateralProperty` fields and `requiredPermissions`.
   - JSDoc comments document fields, schemas, and permissions.
   - A utility generates TypeScript types from `pathRegistry` and `zod` schemas.

8. **Scalability for Microfrontend Architecture**:
   - `pathRegistry`, `zod` schemas, and `NavigationService` types are shared via a `shared-types` package with Module Federation.
   - Miniapps use `NavigationService` via `ShellContext` without direct dependency on the shell.
   - Supports nested routes with typed `appData` and permissions.

9. **Handling Edge Cases**:
   - Undefined `pathKey` throws an error and prevents navigation.
   - Direct page access checks for missing/invalid `appData` or permissions, redirecting to a fallback or displaying an error.
   - Handles multiple source pages with context-specific requirements.

10. **Performance Optimization**:
    - Minimizes context updates for `appData`.
    - Loads `pathRegistry` and `zod` schemas efficiently.
    - Ensures validation and permission checks take <10ms, monitored via logs.

11. **Testing and Validation**:
    - Unit tests verify `NavigationService.navigateTo` for field presence, type correctness, and permission checks.
    - Integration tests ensure navigation respects permissions and passes typed `appData`.
    - A test script validates `pathRegistry` for completeness.
    - Linter warns for undefined `pathKey` or incorrect types.

12. **Zod-Based Validation and Permission Checking**:
    - `pathRegistry` defines types and permissions for each field.
    - `NavigationService` validates `appData` using `zod` schemas and checks permissions.
    - Logs detailed `zod` errors and permission failures.
    - Destination pages assume valid data after `zod` validation.

13. **Visualization of Navigation Flow**:
    - A Mermaid sequence diagram visualizes navigation, including:
      - User interaction to initiate navigation.
      - `NavigationService` validation using `zod` schemas.
      - Permission checking via `ICollateralProperty.permissions`.
      - Storage of `appData` and navigation.
      - Retrieval and validation in the destination miniapp.
      - Error handling for invalid data or permissions.
    - Includes participants: User, MiniApp1, ShellContext, NavigationService, PathRegistry, MiniApp2, Router.

## Notes
- **Navigation Service**: Centralizes navigation logic, improving modularity, maintainability, and testability.
- **Zod Integration**: Validates `ICollateralProperty` with `id`, `status`, `valuations`, `titles`, and `permissions`.
- **Permission Checking**: Ensures secure navigation by checking `ICollateralProperty.permissions`.
- **Refactoring**: Script automates `webPath` conversion with `ICollateralProperty`, `nabValuationId`, and permissions.
- **Microfrontend Compatibility**: Module Federation shares `zod`, types, and permissions.
- **Alignment with Diagram**: The sequence diagram reflects the `NavigationService`.

## Reference: Mermaid Sequence Diagram
The following sequence diagram visualizes the navigation process with `NavigationService`:

```mermaid
sequenceDiagram
    participant User
    participant MiniApp1
    participant ShellContext
    participant NavigationService
    participant PathRegistry
    participant MiniApp2
    participant Router

    User->>MiniApp1: Clicks "Navigate to Path1" button
    MiniApp1->>ShellContext: Access navigationService
    ShellContext-->>MiniApp1: Returns navigationService
    MiniApp1->>NavigationService: Calls navigateTo('path1', { required: { property: { id: "prop123", status: "active", valuations: [100000, 120000], titles: ["title1", "title2"], permissions: ["view-collateral"] }, nabValuationId: "val456" }, optional: { theme: "dark" } })
    
    NavigationService->>PathRegistry: Look up config for 'path1'
    PathRegistry-->>NavigationService: Returns { path: '/path1', requiredFields: [{name: 'property', type: 'ICollateralProperty', zodSchema: ICollateralPropertySchema}, {name: 'nabValuationId', type: 'string'}], requiredPermissions: ["view-collateral", "edit-collateral"] }
    
    NavigationService->>NavigationService: Validate fields (ICollateralPropertySchema.safeParse for property, type for nabValuationId)
    alt Validation passes
        NavigationService->>NavigationService: Check permissions (property.permissions includes "view-collateral")
        alt Permissions granted
            NavigationService->>NavigationService: Log navigation details
            NavigationService->>ShellContext: Store appData
            NavigationService->>Router: Navigate to '/path1'
            
            Router->>MiniApp2: Render MiniApp2
            MiniApp2->>ShellContext: Retrieve appData
            ShellContext-->>MiniApp2: Returns { required: { property: { id: "prop123", status: "active", valuations: [100000, 120000], titles: ["title1", "title2"], permissions: ["view-collateral"] }, nabValuationId: "val456" }, optional: { theme: "dark" } }
            MiniApp2->>MiniApp2: Validate property with ICollateralPropertySchema.safeParse (optional)
            alt Data valid
                MiniApp2->>User: Display property.id, property.status
            else Data invalid
                MiniApp2->>User: Display error (e.g., "Invalid property: Expected string for id")
            end
        else Permissions denied
            NavigationService->>NavigationService: Log error (e.g., "Insufficient permissions")
            NavigationService->>MiniApp1: Throw error
            MiniApp1->>User: Display warning (e.g., "Insufficient permissions")
        end
    else Validation fails
        NavigationService->>NavigationService: Log error (e.g., "Invalid ICollateralProperty")
        NavigationService->>MiniApp1: Throw error
        MiniApp1->>User: Display warning
    end
```