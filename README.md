# GET /users/me - View Profile 

```mermaid
flowchart TD
    UserAction[User clicks 'View Profile'] -->|Authenticated session| ProfileRequest[GET /users/me]
    ProfileRequest -->|Valid session| LoadProfile[Load user data + subscription + keys]
    LoadProfile -->|Data found| ProfileDisplay[Display complete profile]
    LoadProfile -->|Partial data| PartialDisplay[Display profile with missing info notice]
    
    ProfileRequest -->|Invalid session| LoginPrompt[Redirect to login]
    ProfileRequest -->|User not found| ErrorPage[Show 'Profile not found' error]
    
    ProfileDisplay -->|User satisfaction| ProfileActions[Profile action options]
    PartialDisplay -->|User satisfaction| ProfileActions
    
    ProfileActions -->|Edit Profile| EditFlow[Go to PATCH /users/me]
    ProfileActions -->|Manage Keys| KeysFlow[Go to API Keys section]
    ProfileActions -->|Settings| SettingsFlow[Go to account settings]
    
    subgraph Profile Information Displayed
        PersonalInfo[Name, Email, Role, Verification Status]
        SubscriptionInfo[Plan Type, Key Limit, Usage Limit]
        KeysInfo[Current API Keys List]
    end
```

# PATCH /users/me - Update Profile

```mermaid
flowchart TD
    UserAction["User clicks 'Edit Profile'"] -->|From profile page| EditForm["Show profile edit form"]
    EditForm -->|Pre-filled with current data| UserInput["User modifies name field"]
    
    UserInput -->|Clicks 'Save'| ValidateInput["Client-side validation"]
    ValidateInput -->|Invalid format| ShowError["Show validation error"]
    ValidateInput -->|Valid| SubmitUpdate["PATCH /users/me"]
    
    ShowError -->|User corrects| UserInput
    
    SubmitUpdate -->|Valid session + data| UpdateProfile["Update user profile"]
    SubmitUpdate -->|Invalid session| LoginRedirect["Redirect to login"]
    SubmitUpdate -->|Invalid data| ServerError["Show server validation error"]
    
    UpdateProfile -->|Success| ShowSuccess["Show 'Profile updated successfully'"]
    ShowSuccess -->|Auto redirect| UpdatedProfile["Display updated profile"]
    
    ServerError -->|User corrects| UserInput
    
    subgraph Form Fields
        NameField["Name input field"]
        ValidationRules["Name validation rules displayed"]
    end
```

# DELETE /users/me - Delete Account

```mermaid
flowchart TD
    UserAction["User clicks 'Delete Account'"] -->|From settings page| WarningDialog["Show deletion warning dialog"]
    
    WarningDialog -->|"Are you sure?" confirmation| UserDecision{"User confirms deletion?"}
    UserDecision -->|No| CancelAction["Cancel - return to settings"]
    UserDecision -->|Yes| ConfirmDialog["Final confirmation dialog"]
    
    ConfirmDialog -->|"Type DELETE to confirm"| TypeConfirmation["User types DELETE"]
    TypeConfirmation -->|Correct text| SubmitDelete["DELETE /users/me"]
    TypeConfirmation -->|Incorrect text| ConfirmDialog
    
    ConfirmDialog -->|Cancel| CancelAction
    
    SubmitDelete -->|Valid session| DeleteAccount["Account deleted successfully"]
    SubmitDelete -->|Invalid session| LoginRedirect["Redirect to login"]
    
    DeleteAccount -->|Success| LogoutUser["Auto logout + clear cookies"]
    LogoutUser -->|Redirect| GoodbyePage["Show 'Account deleted' message"]
    GoodbyePage -->|After delay| HomePage["Redirect to homepage"]
    
    subgraph Warning Messages
        DataLoss["All data will be permanently deleted"]
        NoRecovery["This action cannot be undone"]
        ImpactInfo["API keys will stop working immediately"]
    end
```

# DELETE /users/me/api-key/:keyId - Revoke API Key

```mermaid

flowchart TD
    UserAction["User clicks 'Delete' on specific key"] -->|From keys list| ConfirmDialog["Show deletion confirmation"]
    
    ConfirmDialog -->|Shows key description| UserDecision{"Confirm key deletion?"}
    UserDecision -->|No| CancelAction["Cancel - return to keys list"]
    UserDecision -->|Yes| SubmitDelete["DELETE /users/me/api-key/:keyId"]
    
    SubmitDelete -->|Valid session + keyId| FindKey["Verify key ownership"]
    SubmitDelete -->|Invalid session| LoginRedirect["Redirect to login"]
    
    FindKey -->|Key not found| KeyNotFound["404 - Key not found error"]
    FindKey -->|Key found + owned| DeleteKey["Remove API key"]
    
    DeleteKey -->|Success| ShowSuccess["Show 'Key deleted successfully'"]
    ShowSuccess -->|Update UI| RefreshList["Refresh keys list"]
    RefreshList -->|Updated list| KeysPage["Display remaining keys"]
    
    KeyNotFound -->|Error message| ErrorDisplay["Show error to user"]
    ErrorDisplay -->|Return| KeysPage
    
    subgraph Deletion Confirmation
        KeyInfo["Show key description and creation date"]
        WarningMsg["Warning about apps using this key"]
        ConfirmBtn["Confirm deletion button"]
        CancelBtn["Cancel button"]
    end


```

# PUT /users/me/password - Change Password

```mermaid
flowchart TD
    UserAction["User clicks 'Change Password'"] -->|From settings page| PasswordForm["Show password change form"]
    
    PasswordForm -->|Three input fields| UserInput["User fills form"]
    UserInput -->|Current + New + Confirm passwords| ClientValidation["Client-side validation"]
    
    ClientValidation -->|Passwords don't match| MatchError["Show 'Passwords don't match'"]
    ClientValidation -->|Weak new password| StrengthError["Show password strength requirements"]
    ClientValidation -->|Valid| SubmitChange["PUT /users/me/password"]
    
    MatchError -->|User corrects| UserInput
    StrengthError -->|User strengthens| UserInput
    
    SubmitChange -->|Valid session| VerifyCurrentPassword["Server verifies current password"]
    SubmitChange -->|Invalid session| LoginRedirect["Redirect to login"]
    
    VerifyCurrentPassword -->|Incorrect current password| CurrentPasswordError["400 - Wrong current password"]
    VerifyCurrentPassword -->|Correct| UpdatePassword["Hash and save new password"]
    
    UpdatePassword -->|Success| PasswordChanged["Show 'Password changed successfully'"]
    PasswordChanged -->|Security notice| SecurityNotice["Recommend updating saved passwords"]
    SecurityNotice -->|Return| SettingsPage["Back to settings"]
    
    CurrentPasswordError -->|Error message| ShowCurrentError["Highlight current password field"]
    ShowCurrentError -->|User retries| UserInput
    
    subgraph Password Form Fields
        CurrentPwd["Current password field"]
        NewPwd["New password field"]
        ConfirmPwd["Confirm new password field"]
        StrengthMeter["Password strength indicator"]
    end
    
    subgraph Security Features
        PasswordRules["Password requirements displayed"]
        StrengthValidation["Real-time strength checking"]
        SecurityTips["Password security tips"]
    end

```

# POST /users/me/request - Submit Logo Request

```mermaid
flowchart TD
    UserAction["User clicks 'Request Logo'"] -->|From dashboard or menu| RequestForm["Show logo request form"]
    
    RequestForm -->|Company URL field| UserInput["User enters company URL"]
    UserInput -->|Clicks 'Submit Request'| ValidateURL["Client-side URL validation"]
    
    ValidateURL -->|Invalid URL format| URLError["Show 'Invalid URL format' error"]
    ValidateURL -->|Valid URL| SubmitRequest["POST /users/me/request"]
    
    URLError -->|User corrects| UserInput
    
    SubmitRequest -->|Valid session + URL| ProcessRequest["Create logo request"]
    SubmitRequest -->|Invalid session| LoginRedirect["Redirect to login"]
    SubmitRequest -->|Invalid URL| ServerValidation["400 - Invalid URL"]
    
    ProcessRequest -->|Success| RequestSubmitted["Show 'Request submitted successfully'"]
    RequestSubmitted -->|Confirmation message| ShowDetails["Display request details"]
    ShowDetails -->|Timeline info| StatusInfo["Show processing timeline"]
    StatusInfo -->|Return option| Dashboard["Return to dashboard"]
    
    ServerValidation -->|Error message| URLError
    
    subgraph Request Form
        URLField["Company URL input field"]
        SubmitBtn["Submit request button"]
        URLHelp["URL format help text"]
    end
    
    subgraph Request Confirmation
        SuccessMsg["Success message"]
        RequestID["Request ID display"]
        Timeline["Expected processing time"]
        ContactInfo["How to follow up"]
    end
    
    subgraph URL Validation
        FormatCheck["URL format validation"]
        DomainCheck["Domain accessibility check"]
        ExistingCheck["Check if already requested"]
    end


```

# POST /users/me/api-key - Generate API Key

```mermaid
flowchart TD
    UserAction["User clicks 'Generate New API Key'"] -->|From API keys page| CheckLimits["Check current key count vs limit"]
    
    CheckLimits -->|At limit| LimitError["Show 'Key limit reached' error"]
    CheckLimits -->|Under limit| ShowForm["Show key generation form"]
    
    LimitError -->|Upgrade suggestion| UpgradePrompt["Suggest plan upgrade"]
    LimitError -->|Delete existing key| ManageKeys["Go to key management"]
    
    ShowForm -->|User enters description| DescriptionInput["Key description input"]
    DescriptionInput -->|Clicks 'Generate'| SubmitRequest["POST /users/me/api-key"]
    
    SubmitRequest -->|Valid session + data| GenerateKey["Create new API key"]
    SubmitRequest -->|Invalid session| LoginRedirect["Redirect to login"]
    SubmitRequest -->|At limit| ServerLimitError["403 - Key limit reached"]
    
    GenerateKey -->|Success| ShowKey["Display new API key"]
    ShowKey -->|One-time display| CopyPrompt["Prompt user to copy key"]
    CopyPrompt -->|Key copied| KeysList["Add to user's keys list"]
    CopyPrompt -->|Security warning| SecurityNotice["Show 'Save this key' warning"]
    
    ServerLimitError -->|Error message| LimitError
    
    subgraph "Key Generation Form"
        DescField["Description field"]
        GenerateBtn["Generate button"]
        FormValidation["Client-side validation"]
    end
    
    subgraph "Security Features"
        OneTimeDisplay["Key shown only once"]
        CopyButton["Copy to clipboard button"]
        SecurityWarning["Security best practices"]
    end


```
