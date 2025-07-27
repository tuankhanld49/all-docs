sequenceDiagram
    participant User
    participant MiniApp1
    participant ShellContext
    participant PathRegistry
    participant MiniApp2
    participant Router

    User->>MiniApp1: Clicks "Navigate to Path1" button
    MiniApp1->>ShellContext: Calls navigate('path1', { required: { user: { id: "123", name: "John" }, orderId: "456" }, optional: { theme: "dark" } })
    
    ShellContext->>PathRegistry: Look up config for 'path1'
    PathRegistry-->>ShellContext: Returns { path: '/path1', requiredFields: [{name: 'user', type: 'IUser', zodSchema: IUserSchema}, {name: 'orderId', type: 'string'}] }
    
    ShellContext->>ShellContext: Validate fields (IUserSchema.safeParse for user, type for orderId)
    alt Validation passes
        ShellContext->>ShellContext: Log navigation details
        ShellContext->>ShellContext: Store appData
        ShellContext->>Router: Navigate to '/path1'
        
        Router->>MiniApp2: Render MiniApp2
        MiniApp2->>ShellContext: Retrieve appData
        ShellContext-->>MiniApp2: Returns { required: { user: { id: "123", name: "John" }, orderId: "456" }, optional: { theme: "dark" } }
        MiniApp2->>MiniApp2: Validate user with IUserSchema.safeParse (optional)
        alt Data valid
            MiniApp2->>User: Display user.name
        else Data invalid
            MiniApp2->>User: Display error (e.g., "Invalid user: Expected string for id")
        end
    else Validation fails
        ShellContext->>ShellContext: Log error (e.g., "Invalid IUser: Expected string for id")
        ShellContext->>MiniApp1: Prevent navigation
        MiniApp1->>User: Display warning
    end