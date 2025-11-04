# Claude Code - iOS Development Guide

## Project Overview

This is an iOS application built with the following specifications:

- **Target iOS Version:** iOS 18+
- **UI Framework:** SwiftUI only (no UIKit)
- **Architecture:** MVVM (Model-View-ViewModel)
- **Dependency Management:** Swift Package Manager (SPM)
- **Database:** RealmSwift (NO CoreData or SwiftData)
- **Concurrency:** async/await (no completion handlers)

- Use Context7 to check up-to-date docs when needed for implementing new libraries or frameworks, or adding features using them.
---

## Code Style & Conventions

Follow these Swift coding standards:

### General Rules
- Use `camelCase` for variables and functions
- Use `PascalCase` for types (classes, structs, enums, protocols)
- Prefer `let` over `var` whenever possible
- Use trailing closures when appropriate
- Use explicit `self` only when required by the compiler
- Maximum line length: **130 characters**
- Organize code sections with `// MARK: -` comments

### Examples
```swift
// MARK: - Properties
private let userManager: UserManager
@State private var isLoading = false

// MARK: - Body
var body: some View {
    VStack {
        // View content
    }
}

// MARK: - Methods
private func fetchData() async {
    // Implementation
}
```

---

## Naming Conventions

Follow these patterns consistently:

| Component | Pattern | Example |
|-----------|---------|---------|
| Views | `[Feature]View` | `UserProfileView`, `SettingsView` |
| ViewModels | `[Feature]ViewModel` | `UserProfileViewModel`, `SettingsViewModel` |
| Models | Noun | `User`, `Post`, `Profile` |
| Managers | `[Feature]Manager` | `UserManager`, `AuthManager` |
| Services | `[Feature]Service` | `NetworkService`, `AuthenticationService` |
| Enums | Noun or Adjective | `UserRole`, `LoadingState` |

---

## Architecture Patterns (Opinionated)

### Navigation
- **Use NavigationCoordinator with NavigationPath** for all navigation logic
- Centralize navigation in a coordinator to keep views clean and testable
- Support deep linking through the coordinator

### Dependency Injection
- **Prefer Environment Objects** for passing dependencies down the view hierarchy
- Use init injection for ViewModels when testing is critical
- Never use global singletons unless absolutely necessary

**Example:**
```swift
@main
struct MyApp: App {
    @StateObject private var userManager = UserManager()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(userManager)
        }
    }
}

// In child views
struct ContentView: View {
    @EnvironmentObject var userManager: UserManager
}
```

### Network Layer
- **Always use a wrapper NetworkService pattern** with async/await
- Never put networking code directly in ViewModels
- Centralize URL construction, headers, and error handling in the service

**Example:**
```swift
class NetworkService {
    func fetch<T: Decodable>(_ endpoint: String) async throws -> T {
        let url = URL(string: "https://api.example.com/\(endpoint)")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode(T.self, from: data)
    }
}
```

### State Management
- Use `@State` for local view state
- Use `@StateObject` for ViewModels owned by the view
- Use `@ObservedObject` for ViewModels passed from parent views
- Use `@EnvironmentObject` for app-wide dependencies

### Error Handling
- Create custom Error enums for different error domains
- Use Result types when appropriate
- Always provide user-friendly error messages

**Example:**
```swift
enum NetworkError: LocalizedError {
    case invalidURL
    case noData
    case decodingFailed
    
    var errorDescription: String? {
        switch self {
        case .invalidURL: return "Invalid URL"
        case .noData: return "No data received"
        case .decodingFailed: return "Failed to decode response"
        }
    }
}
```

---

## Database - RealmSwift

### Rules
- **NEVER use CoreData or SwiftData**
- Use RealmSwift for all local data persistence
- Follow Realm's best practices for thread-safe operations
- Use Realm's property wrappers for reactive updates

### Best Practices
- Always use `@Realm` property wrapper in views
- Perform writes in background threads when possible
- Use Realm's built-in async/await support
- Never pass Realm objects directly between threads

**Example:**
```swift
import RealmSwift

class User: Object, ObjectKeyIdentifiable {
    @Persisted(primaryKey: true) var id: ObjectId
    @Persisted var name: String
    @Persisted var email: String
}
```

---

## Best Practices

### Code Organization
- **No business logic in Views** - Views should only handle UI and user interactions
- **ViewModels contain business logic** - All data manipulation and business rules live here
- **Services handle external operations** - Networking, database, authentication, etc.
- Keep files focused and single-purpose

### Async/Await
- **Always use async/await** - Never use completion handlers
- Use `Task` for creating new async contexts
- Handle errors with do-catch blocks
- Use `@MainActor` for UI updates when necessary

### Security
- **Never hardcode API keys or secrets** - Use environment variables or configuration files
- **Always use HTTPS** for network requests
- Use Keychain for sensitive data (tokens, passwords, credentials)
- Use UserDefaults only for non-sensitive user preferences
- Implement certificate pinning for critical API calls

### Performance
- Use lazy loading for expensive operations
- Profile with Instruments before optimizing
- Avoid premature optimization
- Use SwiftUI's built-in performance tools

---

## Security & Privacy

### Data Storage
- **Keychain:** API tokens, passwords, authentication credentials
- **UserDefaults:** User preferences, app settings (non-sensitive only)
- **Realm:** App data, cached content
- **Never store:** Credit card numbers, SSNs, or highly sensitive PII without encryption

### Network Security
- All network requests must use HTTPS
- Implement proper certificate validation
- Never disable SSL/TLS verification
- Use URLSession with proper security configurations

### Privacy
- Include Privacy Manifest (PrivacyInfo.xcprivacy) for App Store compliance
- Request permissions with clear explanations
- Follow Apple's privacy guidelines
- Be transparent about data collection

---

## Testing

### Test Organization
- Tests location: `[ProjectName]Tests`
- Mirror the main project structure in tests
- Name test files: `[ClassName]Tests.swift`

### Testing Strategy
- **Unit test ViewModels** - Business logic should be fully tested
- **Mock NetworkService** - Create mock implementations for testing
- Test edge cases and error conditions
- Use XCTest framework

**Example:**
```swift
final class UserViewModelTests: XCTestCase {
    var sut: UserViewModel!
    var mockNetworkService: MockNetworkService!
    
    override func setUp() {
        super.setUp()
        mockNetworkService = MockNetworkService()
        sut = UserViewModel(networkService: mockNetworkService)
    }
    
    func testFetchUser() async throws {
        // Test implementation
    }
}
```

---

## Project Structure

Project structure depends on the specific project needs, but generally organize code into logical groups:

```
ProjectName/
├── App/
│   ├── ProjectNameApp.swift
│   └── ContentView.swift
├── Features/
│   ├── Authentication/
│   │   ├── Views/
│   │   ├── ViewModels/
│   │   └── Models/
│   └── UserProfile/
│       ├── Views/
│       ├── ViewModels/
│       └── Models/
├── Services/
│   ├── NetworkService.swift
│   └── AuthenticationService.swift
├── Managers/
│   ├── UserManager.swift
│   └── RealmManager.swift
├── Utilities/
│   ├── Extensions/
│   └── Helpers/
└── Resources/
    ├── Assets.xcassets
    └── Localization/
```

---

## Git Workflow

### Branch Naming Convention
Follow this pattern for all branches:

- `feature/XT-999-brief-description` - For new features
- `bugfix/XT-999-brief-description` - For bug fixes

**Examples:**
- `feature/XT-123-user-authentication`
- `bugfix/XT-456-login-crash`

### Commit Message Format
All commits must follow this format:

```
XT-999: type: Commit message
```

**Ticket Number:** Extract from branch name (e.g., `XT-999`)
**Type:** One of the following:
- `feat` - New feature
- `fix` - Bug fix
- `refactor` - Code refactoring (no functional changes)
- `docs` - Documentation changes
- `test` - Adding or updating tests

**Examples:**
```
XT-123: feat: Add user authentication with biometrics
XT-456: fix: Resolve navigation coordinator memory leak
XT-789: refactor: Improve NetworkService error handling
XT-321: docs: Update README with setup instructions
```

### Commit Process (Manual Approval)
1. **Claude will suggest commits** when appropriate milestones are reached
2. **Claude will generate a commit message** following the format above
3. **Claude will extract the ticket number** from the current branch name
4. **Claude will ask for your approval** before executing the commit
5. **You review and approve** or request changes to the commit message

### When to Commit
Commit after:
- Completing a feature or sub-feature
- Fixing a bug
- Completing a significant refactoring
- Adding tests
- Updating documentation

### Git Commands
Claude will use these git commands (after approval):
```bash
git add <files>
git commit -m "XT-999: feat: Commit message"
git status
git diff
```

**Note:** Claude will NOT automatically push to remote. You maintain control over when to push changes.

---

## GitHub Issues Integration

### Issue Format
When creating Issues, follow this template:

```markdown
Title: XT-XXX: [Brief description]

## Description
[Detailed description of the issue, feature, or bug]

## Tasks
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

## Acceptance Criteria
- Criterion 1
- Criterion 2

## Technical Notes
[Any implementation details, considerations, or constraints]
```

### When Claude Should CREATE Issues

Create new Issues in these scenarios:
1. **Discovering bugs during development** - Only if the bug is NOT related to the current feature or ticket
2. **Identifying technical debt** - Code smells, performance issues, or areas needing refactoring
3. **Suggesting new features/improvements** - Ideas for enhancements or optimizations

**Process:**
- Claude will draft the Issue following the template
- Claude will ask for your approval before creating
- You review and approve or request changes

### When Claude Should UPDATE Issues

Update existing Issues in these scenarios:
1. **Check off completed tasks** - Mark checklist items as done when completed
2. **Add status update comments** - Provide progress updates on ongoing work
3. **Document blockers or issues** - Comment when encountering problems or dependencies
4. **Close completed Issues** - Mark Issues as closed when all work is done

**Process:**
- Claude will automatically check off completed tasks from checklists
- Claude will add comments with progress updates
- Claude will ask before closing an Issue

### Issue Linking in Commits

All commits should reference their related Issue:

```
XT-999: feat: Add user authentication (#123)
```

Where `#123` is the GitHub Issue number.

### Pull Request Workflow

Instead of auto-closing Issues, Claude will:
1. **Ask if you want to create a Pull Request** when work on an Issue is complete
2. **Generate a PR description** that references the Issue
3. **Wait for your approval** before creating the PR
4. Issues will be closed when the PR is merged (using GitHub's linking keywords)

**PR Description Template:**
```markdown
## Description
[Summary of changes]

## Related Issue
Closes #123

## Changes Made
- Change 1
- Change 2
- Change 3

## Testing
- [ ] Unit tests added/updated
- [ ] Manual testing completed
- [ ] No breaking changes
```

---

## Working with Claude Code

### Communication Style
- Be explicit about requirements and constraints
- Ask questions when unclear about specifications
- Suggest improvements and best practices
- Flag potential issues or security concerns early

### Code Review
- Review generated code for adherence to these guidelines
- Check for security vulnerabilities
- Verify proper error handling
- Ensure test coverage for critical paths

### Task Completion
- Complete one task fully before moving to the next
- Ask for clarification if requirements are ambiguous
- Provide progress updates for long-running tasks
- Summarize changes after completing work

---

## Additional Notes

### SwiftUI Previews
- Always include SwiftUI previews for views when possible
- Provide multiple preview configurations (different data states, light/dark mode)
- Use preview data for realistic previews

### Accessibility
- No specific accessibility requirements at this time
- However, follow SwiftUI's default accessibility practices (proper labels, etc.)

### Localization
- Prepare code for localization (use LocalizedStringKey when appropriate)
- Avoid hardcoded strings in UI components

### Dependencies
- Keep dependencies to a minimum
- Only add well-maintained, trusted packages
- Document why each dependency is necessary

---

## Quick Reference

### Preferred Tools & Frameworks
✅ SwiftUI
✅ async/await
✅ RealmSwift
✅ Swift Package Manager
✅ NavigationCoordinator with NavigationPath
✅ Environment Objects

### Avoid
❌ UIKit (use SwiftUI only)
❌ CoreData / SwiftData
❌ Completion handlers (use async/await)
❌ Hardcoded secrets
❌ Business logic in Views
❌ Global singletons (except when necessary)

