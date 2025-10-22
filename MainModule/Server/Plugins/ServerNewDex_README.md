# ServerNewDex - Server-Side Mode

## Overview
The ServerNewDex has been enhanced to provide full server-side access to all Roblox services, including server-only containers like ServerScriptService, ServerStorage, and ServerScriptService.

## Changes Made

### Server-Side Enhancements

1. **Instance Reference Caching System**
   - Server maintains a cache of instance references per player
   - Instances are assigned unique IDs (prefixed with `server_`)
   - Client can reference server-only instances using these IDs

2. **New Server Actions**
   The following actions have been added to the RemoteFunction handler:

   - **`getservice`** - Get a service by name
     ```lua
     Usage: RemoteFunction:InvokeServer("getservice", serviceName)
     Returns: {Success: bool, InstanceId: string, Name: string, ClassName: string, IsService: bool}
     ```

   - **`getchildren`** - Get children of any instance (including server-only)
     ```lua
     Usage: RemoteFunction:InvokeServer("getchildren", instanceRef)
     Returns: {Success: bool, Children: {{InstanceId: string, Name: string, ClassName: string}...}}
     ```

   - **`getproperty`** - Get property value of any instance
     ```lua
     Usage: RemoteFunction:InvokeServer("getproperty", instanceRef, propertyName)
     Returns: {Success: bool, Value: any, Type: string, InstanceId: string?}
     ```

   - **`getgame`** - Get the game (DataModel) root
     ```lua
     Usage: RemoteFunction:InvokeServer("getgame")
     Returns: {Success: bool, InstanceId: string, Name: string, ClassName: string}
     ```

   - **`getdescendants`** - Get all descendants of an instance
     ```lua
     Usage: RemoteFunction:InvokeServer("getdescendants", instanceRef)
     Returns: {Success: bool, Descendants: {{InstanceId: string, Name: string, ClassName: string, Parent: string}...}}
     ```

3. **Updated Existing Actions**
   All existing actions (destroy, duplicate, copy, paste, setproperty, etc.) now support cached server-side instance references. They can accept either:
   - Direct instance references (for client-visible instances)
   - String IDs starting with `server_` (for server-only instances)

4. **Automatic Cleanup**
   - Instance cache is automatically cleared when players leave
   - Prevents memory leaks

## Using Server-Side Services

### Example: Accessing ServerScriptService

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local DexEvent = ReplicatedStorage:WaitForChild("NewDex_Event")

-- Get ServerScriptService
local result = DexEvent:InvokeServer("getservice", "ServerScriptService")
if result.Success then
    print("ServerScriptService ID:", result.InstanceId)

    -- Get its children
    local children = DexEvent:InvokeServer("getchildren", result.InstanceId)
    if children.Success then
        for _, child in ipairs(children.Children) do
            print("Child:", child.Name, child.ClassName)
        end
    end
end
```

### Example: Accessing ServerStorage

```lua
local result = DexEvent:InvokeServer("getservice", "ServerStorage")
if result.Success then
    -- Get all descendants
    local descendants = DexEvent:InvokeServer("getdescendants", result.InstanceId)
    if descendants.Success then
        print("Found", #descendants.Descendants, "descendants in ServerStorage")
    end
end
```

## Client-Side Integration

To fully integrate these server-side capabilities into the Dex UI, the client-side Explorer code would need to be modified to:

1. Detect when trying to access server-only services
2. Call the appropriate server actions instead of local methods
3. Display the returned data in the explorer tree
4. Use cached instance IDs for subsequent operations

## Security Notes

- All server actions verify that the player is authorized via the `ServerNewDex.Authorized` table
- Unauthorized access attempts will kick the player
- The server has full control over what data is exposed and what operations are allowed

## Compatibility

- Fully backward compatible with existing dex functionality
- Client-visible instances continue to work as before
- New server actions are optional and only needed for server-only access
