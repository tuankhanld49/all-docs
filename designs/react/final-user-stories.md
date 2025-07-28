### Revised Solution Summary

**Objective**: Provide a centralized navigation solution for a microfrontend application that supports generic `appData` types, validates data with `zod`, and performs permission checks based on path names.

#### Key Components
1. **NavigationService**:
   - Centralizes navigation logic: path lookup, `appData` validation, and conditional permission checks.
   - Uses `pathRegistry` to map `pathKey` to `path`, `requiredFields`, and optional `requiredPermissions`.
   - Validates `appData` using `zod` schemas specific to each path’s `requiredFields`.
   - Calls `EntitlementService.checkPermissions` only for paths with `requiredPermissions`.
   - Stores `appData` in `ShellContext` and navigates via `react-router-dom`.
   - Throws errors for invalid paths, data, or permissions, displayed in `MiniApp1`.

2. **EntitlementService**:
   - Handles permission checks with `checkPermissions(path: string, requiredPermissions: string[])`.
   - Uses a mock permission store (`userPermissionsByPath`) for path-based validation, pending your permission source details.
   - Throws errors for insufficient permissions (e.g., “Insufficient permissions for path /target-path”).

3. **Shared Types and Zod Schemas**:
   - `appData` is validated with path-specific `zod` schemas defined in `pathRegistry` (e.g., `ICollateralPropertySchema` for paths requiring `property`).
   - Example schema for `ICollateralProperty` (as a sample `appData` type):
     ```typescript
     z.object({
       id: z.string(),
       status: z.string(),
       valuations: z.array(z.unknown()),
       titles: z.array(z.unknown()),
     })
     ```
   - `PathConfig`:
     ```typescript
     type PathConfig = {
       path: string;
       requiredFields: string[];
       requiredPermissions?: string[];
       schema?: z.ZodSchema; // For path-specific appData validation
     }
     ```
   - `pathRegistry` maps `pathKey` to `PathConfig`, including schemas for `appData`.

4. **ShellContext**:
   - Provides `navigationService` and `entitlementService` via `useMemo`.
   - Stores generic `appData` for access by destination miniapps.

5. **Miniapps**:
   - **MiniApp1**: Calls `navigationService.navigateTo` with `pathKey` and `appData`, displays errors (e.g., invalid data, permissions).
   - **MiniApp2**: Retrieves `appData` from `ShellContext`, validates with path-specific schema, and displays details or errors.

6. **Microfrontend Compatibility**:
   - Module Federation shares `NavigationService`, `EntitlementService`, and `shared-types`.
   - Supports nested routes and generic `appData` types.

#### Key Features
- **Generic appData**: Supports any data type in `appData`, validated by path-specific `zod` schemas.
- **Path-Based Permissions**: `checkPermissions` uses `path` and `requiredPermissions`, skipping checks for public paths.
- **Zod Validation**: Ensures type safety for `appData` with detailed error messages.
- **Error Handling**: User-friendly warnings for invalid data or permissions.
- **Performance**: Validation and permission checks <10ms; skips checks for paths without `requiredPermissions`.
- **Testability**: Unit tests for services cover valid/invalid cases.

### Updated User Stories

Below are revised user stories that reflect the generic `appData` approach, with `ICollateralProperty` as an example type, and permission checks tied to path names.



# User Stories for Navigation Solution with Generic appData

## User Story 1: Centralized Navigation with Flexible appData
**As a** developer,  
**I want** a `NavigationService` to handle navigation logic for any `appData` type, including path lookup, data validation, and conditional permission checks,  
**so that** navigation is consistent, maintainable, and supports diverse data types across microfrontends.

**Acceptance Criteria**:
- `NavigationService` is a class with injectable dependencies: `navigate`, `setAppData`, and `entitlementService`.
- It looks up `pathKey` in `pathRegistry` to get `path`, `requiredFields`, optional `requiredPermissions`, and an optional `zod` schema.
- It validates `appData` fields (e.g., `property`, `nabValuationId`) using the path-specific `zod` schema (e.g., `ICollateralPropertySchema` for paths requiring `property`).
- It calls `EntitlementService.checkPermissions` only for paths with `requiredPermissions`.
- It stores `appData` in `ShellContext` and navigates using `react-router-dom`.
- Errors for invalid paths, data, or permissions are thrown and displayed in the source miniapp.
- Unit tests cover navigation with different `appData` types (e.g., `ICollateralProperty`, other types).

## User Story 2: Path-Based Permission Checks with EntitlementService
**As a** developer,  
**I want** an `EntitlementService` to validate permissions based on the path name and required permissions,  
**so that** only authorized users access restricted paths, ensuring security and modularity.

**Acceptance Criteria**:
- `EntitlementService` provides `checkPermissions(path: string, requiredPermissions: string[])` to validate permissions for a path.
- Permissions are checked against a store (e.g., user session or mock `userPermissionsByPath`).
- Returns `true` if all `requiredPermissions` are present, else throws an error (e.g., “Insufficient permissions for path /target-path”).
- Only invoked for paths with non-empty `requiredPermissions` in `pathRegistry`.
- Errors are propagated to `NavigationService` and displayed in the source miniapp.
- Injectable and testable, with unit tests for valid/invalid permissions.

## User Story 3: Type-Safe appData Validation with Zod
**As a** developer,  
**I want** to validate generic `appData` using path-specific `zod` schemas,  
**so that** data is type-safe and errors are caught early with detailed messages.

**Acceptance Criteria**:
- `pathRegistry` includes an optional `schema` field (e.g., `ICollateralPropertySchema` for paths requiring `property`).
- Example schema for `ICollateralProperty`:
  ```typescript
  z.object({
    id: z.string(),
    status: z.string(),
    valuations: z.array(z.unknown()),
    titles: z.array(z.unknown()),
  })
  ```
- `NavigationService` validates `appData` using the path’s `schema` and checks `requiredFields`.
- `nabValuationId` is validated as a `string` if required.
- Validation errors provide detailed messages (e.g., “Expected string for id”).
- Destination miniapp re-validates `appData` from `ShellContext` using the same schema.
- Schemas are shared via Module Federation in `shared-types`.

## User Story 4: Microfrontend Compatibility with Module Federation
**As a** developer,  
**I want** to share `NavigationService`, `EntitlementService`, and schemas across microfrontends using Module Federation,  
**so that** miniapps access consistent navigation and validation logic.

** acceptance Criteria**:
- `shared-types` package exports `zod` schemas and service types.
- `NavigationService` and `EntitlementService` are shared via Module Federation.
- Webpack configs expose services and types with `zod` as a singleton.
- Miniapps access `navigationService` and `appData` via `ShellContext`.
- Supports nested routes and generic `appData` types.

## User Story 5: User-Friendly Error Handling
**As a** user,  
**I want** clear error messages for navigation failures due to invalid data or permissions,  
**so that** I understand why access is denied and can act accordingly.

**Acceptance Criteria**:
- `MiniApp1` catches errors from `NavigationService` (e.g., invalid path, data, permissions).
- Displays user-friendly warnings (e.g., “Invalid data provided” or “You lack permission to view this page”).
- `MiniApp2` validates `appData` and displays errors if invalid (e.g., “Failed to load details”).
- Error messages are consistent across miniapps using a shared utility or component.

## User Story 6: Efficient Navigation Performance
**As a** developer,  
**I want** navigation, validation, and permission checks to be fast,  
**so that** users experience minimal latency and the application scales.

**Acceptance Criteria**:
- Validation and permission checks complete in <10ms for typical cases.
- Permission checks are skipped for paths without `requiredPermissions`.
- `pathRegistry` lookups are optimized (e.g., static or cached).
- `zod` schemas are reused for efficiency.
- Performance is verified with tests under load.

## User Story 7: Support for Public and Restricted Paths
**As a** user,  
**I want** to access public paths without permission checks and restricted paths only with valid permissions,  
**so that** the application is secure and accessible as intended.

**Acceptance Criteria**:
- `pathRegistry` supports paths with optional `requiredPermissions` (e.g., `public-path` has none, `target-path` requires `view-collateral`).
- Public paths skip `checkPermissions`.
- Restricted paths enforce permission checks via `EntitlementService`.
- Navigation to restricted paths fails with clear errors if permissions are insufficient.



### Key Updates
- **Generic appData**: User stories emphasize that `appData` can be any type (e.g., `ICollateralProperty` as an example), validated by path-specific `zod` schemas in `pathRegistry`.
- **Path-Based Permissions**: `checkPermissions` uses `path` and `requiredPermissions`, independent of `appData`.
- **Flexibility**: `pathRegistry` includes a `schema` field to support diverse `appData` types per path.
- **No API**: Removed API assumptions for permissions, using a mock store until you clarify the permission source.

### Implementation Notes
- **EntitlementService**:
  ```typescript
  class EntitlementService {
    checkPermissions(path: string, requiredPermissions: string[]): boolean {
      const userPermissions = userPermissionsByPath[path] || [];
      if (!requiredPermissions.every((perm) => userPermissions.includes(perm))) {
        throw new Error(`Insufficient permissions for path ${path}`);
      }
      return true;
    }
  }
  ```
- **NavigationService**:
  ```typescript
  class NavigationService {
    navigateTo(pathKey: string, appData: any) {
      const config = pathRegistry[pathKey];
      if (!config) throw new Error(`Invalid pathKey: ${pathKey}`);
      config.requiredFields.forEach((field) => {
        if (!appData[field]) throw new Error(`Missing required field: ${field}`);
      });
      if (config.schema) {
        const parsed = config.schema.safeParse(appData);
        if (!parsed.success) throw new Error(`Invalid appData: ${parsed.error.message}`);
      }
      if (config.requiredPermissions?.length) {
        this.entitlementService.checkPermissions(config.path, config.requiredPermissions);
      }
      this.setAppData(appData);
      this.navigate(config.path);
    }
  }
  ```
- **pathRegistry Example**:
  ```typescript
  const pathRegistry: Record<string, PathConfig> = {
    'target-path': {
      path: '/target-path',
      requiredFields: ['property', 'nabValuationId'],
      requiredPermissions: ['view-collateral'],
      schema: ICollateralPropertySchema,
    },
    'public-path': {
      path: '/public-path',
      requiredFields: ['data'],
      schema: z.object({ data: z.string() }), // Example for another appData type
    },
  };
  ```

### Questions for You
1. **Permission Source**: How are permissions for paths determined (e.g., user session, API)? This will refine Story 2.
2. **appData Types**: Can you provide examples of other `appData` types beyond `ICollateralProperty` to define schemas for Story 3?
3. **Path Examples**: Share a sample `pathRegistry` or specific paths for Story 7.
4. **Additional Stories**: Need stories for refactoring, logging, or other features?
5. **Async Needs**: Should `checkPermissions` be async (e.g., for API calls)?
