sequenceDiagram
    participant User
    participant MiniApp1
    participant ShellContext
    participant PathRegistry
    participant MiniApp2
    participant Router

    User->>MiniApp1: Clicks "Navigate to Path1" button
    MiniApp1->>ShellContext: Calls navigate('path1', { required: { userId, orderId }, optional: { theme } })
    
    ShellContext->>PathRegistry: Look up config for 'path1'
    PathRegistry-->>ShellContext: Returns { path: '/path1', requiredFields: ['userId', 'orderId'] }
    
    ShellContext->>ShellContext: Validate required fields (userId, orderId)
    alt Validation passes
        ShellContext->>ShellContext: Log navigation details
        ShellContext->>ShellContext: Store appData under '/path1'
        ShellContext->>Router: Navigate to '/path1'
        
        Router->>MiniApp2: Render MiniApp2 for '/path1'
        MiniApp2->>ShellContext: Retrieve appData for '/path1'
        ShellContext-->>MiniApp2: Returns { required: { userId, orderId }, optional: { theme } }
        MiniApp2->>MiniApp2: Validate required fields
        alt Data valid
            MiniApp2->>User: Display page with data (e.g., userId)
        else Data missing
            MiniApp2->>User: Display error (e.g., "Missing userId")
        end
    else Validation fails
        ShellContext->>ShellContext: Log error (e.g., "Missing required fields")
        ShellContext->>MiniApp1: Prevent navigation, show warning
        MiniApp1->>User: Display error message (optional)
    end


Explanation of the Diagram
Participants:
User: The end-user interacting with the application.
MiniApp1: The source miniapp where navigation is initiated.
ShellContext: The shell’s context managing appData and the navigate function.
PathRegistry: The centralized registry defining paths and their appData requirements.
MiniApp2: The destination miniapp (e.g., for /path1).
Router: The react-router-dom router handling navigation.
Flow:
The user clicks a button in MiniApp1 to navigate to path1.
MiniApp1 calls the navigate function in ShellContext with pathKey (path1) and appData.
ShellContext queries PathRegistry to get the configuration for path1 (e.g., required fields: userId, orderId).
ShellContext validates the provided appData against the required fields.
If validation passes:
ShellContext logs the navigation details, stores appData, and triggers navigation via the Router.
Router renders MiniApp2 for /path1.
MiniApp2 retrieves appData from ShellContext and validates it.
If valid, MiniApp2 displays the page; otherwise, it shows an error.
If validation fails:
ShellContext logs an error and prevents navigation.
MiniApp1 may display a warning to the user.
Key Features Visualized:
Use of pathRegistry for path and data requirements.
Validation at both navigation and rendering stages.
Error handling and logging for debugging.
Data storage in ShellContext for access by destination pages.