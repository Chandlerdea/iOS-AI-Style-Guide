# Style Guide
This guide specifies the how iOS apps should be architected and written in Swift. Use the naming conventions specified in this document; not your own. Use this as a reference for writing Swift code with the style and conventions specified in this document.

## Features
Each screen should have one view paired with a view model. Start with one view/view model pair and only create children when the child view and view model are reused on a separate screen. Both the view and view model should share the same prefix. For example, if the app has a profile screen, there would be a `ProfileViewModel.swift` file containing a `ProfileViewModel` class and a `ProfileView.swift` file containing a `ProfileView` struct. Details for how you should write views and view models are in the following sections.

## View Models
View models should be classes marked as `final` using the `@MainActor` and `@Observable` Swift Macros. Only non-private functions should exist in the class declaration. Any private functions should exist in a `private` `extension` of the class in the same file. 

### Properties and functions
Any properties should be `let` if immutable, or `private(set) var` if mutable. Any `@Environment` dependencies from the view should be passed into the functions that need them, not in the class initializer. 

### Swift Concurrency
Functions in view models should avoid creating local `Tasks` in favor of being `async` and be called from a `Task` initialized at the call site in the view. If you need to create a local `Task` in the view model, it should be for an asynchronous task that must be done on a global actor to avoid blocking the main actor

Using the previous profile example, here's what the `ProfileViewModel` would look like:

```swift
@Observable @MainActor
final class ProfileViewModel {
    let user: User
    private(set) var posts: [Post] = []
    private(set) var subscription: Subscription?
    private(set) var errorMessage: String?
    private var globalActorAsyncWork: Task<Void, Never>?

    init(user: User) {
        self.user = user
 }

    func onTask(apiClient: APIClient) async {
        posts = await apiClient.posts(for: user.id)
        subscription = await apiClient.fetchSubscriptions().first
        doSomeAsynchronousWork(apiClient: apiClient)
    }

    func signOutButtonTapped(apiClient: apiClient) async {
        do {
            await apiClient.signOut()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}

private extension ProfileViewModel {
    func doSomeAsynchronousWork(apiClient: APIClient) {
        globalActorAsyncWork = Task {
            for await value in apiClient.someAsyncStream {
                // Do something with teh value
           }
        }
    }
}
```

## Views
Views should use Apple's `SwiftUI` framework. Each view should have a `private` property of its corresponding view model. If the view model takes an argument in its initializer, pass the dependency via the view's initializer and instantiate the view model in the view's initializer. If the view model does not have an initializer in the class, initialize it in the property declaration in the view. Views should have at least one preview wrapped in a #if DEBUG #endif statement.

### SwiftUI Environment
Global dependencies should use SwiftUI's `@Environment` property wrapper to create dependencies that are passed down the view hierarchy via the environment system.

Using the previous profile example, here's what the `ProfileView` would look like:
```swift
struct ProfileView: View {
    @State var viewModel: ProfileViewModel
    @Environment(\.apiClient) private var apiClient

    init(user: User) {
        self.viewModel = ProfileViewModel(user: user)
    }

    var body: some View {
        NavigationStack {
            VStack {
                // other views
            }
            .navigationTitle(user.displayName)
            .task {
                await viewModel.onTask(apiClient: apiClient)
            }
        }
    }
}
```

## Clients
Clients are types that provide access to external APIs. These APIs could be Apple or third-party frameworks that we cannot mock or web services communicated with over the network. These types are also referred to as "Services," but for our purposes, you should refer to them as "Clients." They should be used when view models or other objects need access to resources or other APIs that cannot be mocked out of the box. Implement cients with the suffix "Clent" (i.e., APIClient) using protocol witnesses. Protocol witnesses are structs that only use closure variables in place of functions. So, each property will be a `var` with the value being a Swift closure. Do not create a protocol; instead, use a struct. The closure properties should also be `@Senable`. Here is an example of a protocol witness:

```swift
struct ProtocolWitnessExample {
    var doSomethingWithOneArgument: @Sendable (Int) async throws -> Int
    var doAnotherThingWithNoArguments: @Sendable () -> Void
}
```
Clients should conform to the Senable protocol and not be isolated to any actor. Each client should have a declaration file (i.e., APIClient.swift) that contains the type declaration and a static mock function, with a correponding argument for each property. Do not create a separate type for mocks. The mock function should have default values for all arguments that print a warning that a the default argument of a mock was called and returns a placeholder value.

Here's an example of a client implementation:

```swift
struct APIClient {
    var getUser: (ID) async throws -> User
    var getPost: (ID) async throws -> Post
    var postContent: (Data) async throws -> Post
}

extension APIClient {
    static func mock(
        getUser: (ID) async throws -> User = { _ in 
            print("A default value was called for \(#function) in \(#fileID) line \(#line)")
            return User.placeholder
        },
        getPost: (ID) async throws -> Post = { _ in
            print("A default value was called for \(#function) in \(#fileID) line \(#line)")
            return Post.placeholder
        },
        postContent: (Data) async throws -> Post = { _ in
            print("A default value was called for \(#function) in \(#fileID) line \(#line)")
            return Post.placeholder
        }
    ) {
        init(
            getUser: getUser,
            getPost: getPost,
            postContent: postContent
        )
    }
}
```

### Live implementation

There should also be a "SomeClient+Live.swift" file in the same directory that contains a static variable named "live" that creates an instance of the client inside of an extension on the client. Do not create a separate type for the live implementation. This extension will contain the live ilementation of the client that communicates with
the extenrnal APIs. Do not create an intermediary object that communicates with the API called by the client. Here's an example of a live implementation of the `APIClient` example above:

```swift
extension APIClient {
    static var live: Self {
        init(
            getUser: { id in
                let request = URLRequest(url: URL(string: "your/web/service")!)
                let (data, response) = try await URLSession.shared.dataTask(with: request)
                // process the request
                return user
             },
            getPost: { id in
                let request = URLRequest(url: URL(string: "your/web/service")!)
                let (data, response) = try await URLSession.shared.dataTask(with: request)
                // process the request
                return post
            },
            postContent: { content in
                var request = URLRequest(url: URL(string: "your/web/service")!)
                request.httpBody = content
                request.httpMethod = "POST"
                let (data, response) = try await URLSession.shared.dataTask(with: request)
                // process the request
                return post
            }
        )
    }
}
```
Use SwiftUI's `EnvironmentKeys` and `EnvironmentValues` protocols to expose Clients to SwifUI's `@Environment` property wrapper. The conformance should exist at the end of the client's declaration file. The `static var defaultValue` property of `EnvironmentKeys` should call `mock`, not `live` when conforming a client.

## Things to avoid in general

### Anti-patterns
Do not implement anti-patterns. This includes:
- Singletons
- Global variables
- God obejcts
    - Examples:
        - Large classes that have multiple responsibilities
- Objects that are open to modification but closed to extension
    - Instead, objects should be closed to modifiecation but open to extension
- Global mutable state

### Miscelanious
- Do not hardcode API keys
- Avoid writing code that could lead to a crash
    - Force unwrapped
    - Array subscripts without checking if it is out of bounds
