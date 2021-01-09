# Thanks All!

I no longer have the time, energy or interest to maintain this project, so I'm going to be archiving it.

As an alternative to jira-connector, try: https://github.com/MrRefactoring/jira.js

# JavaScript Jira API Wrapper for NodeJS

Node.JS module which provides easy-to-use access to the Jira REST API.

## Important Change in Version 1.0.0

Starting with version 1.0.0, all functions and parameters utilize `xxxId` instead of `xxxID`.
Prior to this update, this naming convention was inconsistent and easily confused.

Please update your code accordingly.

## Installation

Install with [npm](http://npmjs.org/):

```bash
    $ npm install jira-connector
```

Install via Git clone:

```bash
    $ git clone https://github.com/floralvikings/jira-connector.git
    $ cd jira-connector
    $ npm install
```

## Documentation

The documentation for jira-connector can be found in the source; If you'd like to view
the source as an HTML document, you can use [JSDoc](http://usejsdoc.org/) to generate these
pages. Simply run:

```bash
    $ jsdoc -c jsdoc.json
```

In the jira-connector source directory. This will create a `docs` directory, containing the HTML
markup for the docs.

Also, the [official Jira API docs](https://docs.atlassian.com/jira/REST/latest/) are very useful; many of
the functions in jira-connector use the exact same format as the request bodies of these endpoints.

## Example

Retrieve an issue from Jira, and print its summary to the console.

```javascript
// ES5
var JiraClient = require("jira-connector");

// ES6 or Typescript
import JiraClient from "jira-connector";

// Initialize
var jira = new JiraClient({
  host: "jenjinstudios.atlassian.net",
  strictSSL: true // One of optional parameters
});

// ES5
jira.issue.getIssue(
  {
    issueKey: "JWR-19"
  },
  function(error, issue) {
    console.log(issue.fields.summary);
  }
);

// ES6
jira.issue
  .getIssue((issueKey: "JWR-19"))
  .then(issue => {
    console.log(issue.fields.summary);
  })
  .catch(error => throw error);

// ES8 or Typescript. NodeJS 7.6.0 or higher
const issue = await jira.issue.getIssue({ issueKey: "JWR-19" });
```

First, the `JiraApi` class is imported from the `jira-connector` module. This class
provides access to the Jira REST endpoints, organized into related classes.

The `issue` property of the `JiraApi` class is used to retrieve and modify Jira Issues.

All of the methods in the jira-connector API classes take two arguments; the `opts` and `callback`.

- The `opts` argument specifies the options that will be used when communicating with the Jira API. For a detailed
  list of options, see the documentation for the method into which you are passing the options.
- The `callback` argument should be a function expecting two arguments; and error, and the results of the API
  request.

## Authentication

Depending on the Jira instance to which you are connecting, authentication may or may not be required
for various API calls.

jira-connector supports two forms of authentication:

### Basic Authentication (Deprecated)

[Deprecated](https://confluence.atlassian.com/cloud/deprecation-of-basic-authentication-with-passwords-for-jira-and-confluence-apis-972355348.html). It will require you to
provide a username and password each time you connect to the
Jira instance. However, jira-connector supports it for users who are unable to use OAuth.

Example:

```javascript
var JiraClient = require("jira-connector");

var jira = new JiraClient({
  host: "jenjinstudios.atlassian.net",
  basic_auth: {
    username: "SirUserOfName",
    password: "Password123"
  }
});
```

### Basic Authentication With API Token

This is not recommended; it will require you to provide a email and [api_token](https://confluence.atlassian.com/cloud/api-tokens-938839638.html) each time you connect to the
Jira instance. However, jira-connector supports it for users who are unable to use OAuth.

Example:

```javascript
var JiraClient = require("jira-connector");

var jira = new JiraClient({
  host: "jenjinstudios.atlassian.net",
  basic_auth: {
    email: "email@email.com",
    api_token: "api-token"
  }
});
```

### Basic Authentication (Base64)

Also not recommended, but slightly better than the above; it will require you to provide a Base64 encoded username
and password or email and api_token (a Base64 encoding in the format of "username:password", or "email:api_token") each time you connect to the Jira instance.

More examples [here](https://developer.atlassian.com/jiradev/jira-apis/jira-rest-apis/jira-rest-api-tutorials/jira-rest-api-example-basic-authentication).

Example:

```javascript
var JiraClient = require("jira-connector");

var jira = new JiraClient({
  host: "jenjinstudios.atlassian.net",
  basic_auth: {
    base64: "U2lyVXNlck9mTmFtZTpQYXNzd29yZDEyMw=="
  }
});

// Base64 encoding of 'SirUserOfName:Password123' (for legacy server version) or 'email:api_token' (jira cloud)
```

### OAuth Authentication

This should be the preferred method of authentication; it is more secure and does not require disclosing
your password.

However, setting up OAuth access in Jira can be somewhat complicated; first the Jira administrator must create
an Application Link; for instructions on how to do this, see
[Linking to Another Application](https://confluence.atlassian.com/display/JIRA/Linking+to+Another+Application).

[This example](https://developer.atlassian.com/display/JIRADEV/JIRA+REST+API+Example+-+OAuth+authentication) may also
be helpful in configuring OAuth Access.

Once the Application Link has been created, you will need the private key that corresponds to the public key used to
create the link, and the consumer key that was provided when the link was created.

Once you have this data, you will need to generate an OAuth token and secret for your account; jira-connector provides
helper functions for exactly this purpose:

```javascript
var JiraClient = require("jira-connector");

JiraClient.oauth_util.getAuthorizeURL(
  {
    host: "jenjinstudios.atlassian.net",
    oauth: {
      consumer_key: "your-consumer-key",
      private_key: "-----BEGIN RSA PRIVATE KEY-----\n" + "SomePrivateKeyHere\n" + "-----END RSA PRIVATE KEY-----"
    }
  },
  function(error, oauth) {
    console.log(oauth);
  }
);
```

This will output something similar to the following:

```javascript
{
    url: 'https://jenjinstudios.atlassian.net/plugins/servlet/oauth/authorize?oauth_token=some-token-here',
    token: 'some-token-here',
    token_secret: 'some-secret-here'
}
```

You can then visit the specified URL, which will display a page asking you to allow or deny the request for access.
Allowing access will display a verifier code. Once you have this code, you can swap out your current OAuth token
for an Access Token with all the permissions of your account; jira-connector provides a function to help with this:

```javascript
var JiraClient = require("jira-connector");

JiraClient.oauth_util.swapRequestTokenWithAccessToken(
  {
    host: "jenjinstudios.atlassian.net",
    oauth: {
      token: "your-oauth-token",
      token_secret: "your-token-secret",
      oauth_verifier: "verifier-code-from-jira",
      consumer_key: "your-consumer-key",
      private_key: "-----BEGIN RSA PRIVATE KEY-----\n" + "SomePrivateKeyHere\n" + "-----END RSA PRIVATE KEY-----"
    }
  },
  function(error, accessToken) {
    console.log(accessToken);
  }
);
```

This will query Jira for an Access Token, which will then be printed to the screen. Finally, you're ready to access
Jira with OAuth!

```javascript
var JiraClient = require("jira-connector");

var jira = new JiraClient({
  host: "jenjinstudios.atlassian.net",
  oauth: {
    consumer_key: "your-consumer-key",
    private_key: "-----BEGIN RSA PRIVATE KEY-----\n" + "SomePrivateKey\n" + "-----END RSA PRIVATE KEY-----",
    token: "your-access-token",
    token_secret: "your-token-secret"
  }
});

// Jira is now authenticated with your account!
```

### JWT Authentication

JWT (JSON Web Token) for [authenticate apps](https://developer.atlassian.com/cloud/jira/platform/authentication-for-apps/).

- `iss` - The Jira app key. Can be found in the [app descriptor](https://developer.atlassian.com/cloud/jira/platform/app-descriptor/).
- `secret` - The connected shared secret as stored by the app during the installation handshake.
- `expiry_time_seconds` - Optional. Token expiry time in seconds (default 180 seconds).

Example:

```javascript
const JiraClient = require("jira-connector");

const jira = new JiraClient({
  host: "jenjinstudios.atlassian.net",
  jwt: {
    iss: "atlassian-connect-app",
    secret: "your-jwt-secret"
  }
});
```

### Cookie Jar

You can also use a Cookie Jar for your request. It could be an easier way to prompt for a login only once, without the
pain of setting up an OAuth method.

For example, using `tough-cookie-filestore`:

```javascript
var JiraClient = require("jira-connector");
var FileCookieStore = require("tough-cookie-filestore");

var request = require("request");
var path = require("path");

var jar = request.jar(new FileCookieStore(path.join(__dirname, "cookies.json")));

// For the first connection
var jira = new JiraClient({
  host: "jenjinstudios.atlassian.net",
  basic_auth: {
    username: "SirUserOfName",
    password: "Password123"
  },
  cookie_jar: jar
});

// For the following connections
var jira = new JiraClient({
  host: "jenjinstudios.atlassian.net",
  cookie_jar: jar
});
```

In this example, all your cookies are save in a file, `cookies.json`. Currently, the file **MUST** exist, it's a
limitation from `tough-cookie-filestore`...

You can now only use the Cookie Jar for all the following request, as long as the file exists and the cookie
is still valid!

## Supported API Calls
<details>
  <summary>applicationProperties (/rest/api/2/application-properties)</summary>

  - getProperties
  - setProperty
</details>
<details>
  <summary>attachment (/rest/api/2/attachment)</summary>

  - getAttachment
  - deleteAttachment
  - getGlobalAttachmentMetadata
</details>
<details>
  <summary>auditing (/rest/api/2/auditing)</summary>

  - getAudits
  - createAudit
</details>
<details>
  <summary>avatar (/rest/api/2/avatar) (Untested; use at your own peril)</summary>

  - getAvatars
  - createTemporaryAvatar
  - cropTemporaryAvatar
</details>
<details>
  <summary>backlog (/rest/agile/1.0/backlog)</summary>

  - moveIssuesToBacklog
  - moveIssuesToBacklogForBoard
</details>
<details>
  <summary>board (/rest/agile/1.0/board)</summary>

  - getAllBoards
  - createBoard
  - getBoardByFilterId
  - getBoard
  - deleteBoard
  - getIssuesForBacklog
  - getConfiguration
  - getEpics
  - getIssuesWithoutEpic
  - getIssuesForEpic
  - getFeaturesForBoard
  - toggleFeatures
  - getIssuesForBoard
  - moveIssuesToBoard
  - getProjects
  - getProjectsFull
  - getBoardPropertyKeys
  - getBoardProperty
  - setBoardProperty
  - deleteBoardProperty
  - getAllQuickFilters
  - getQuickFilter
  - getReportsForBoard
  - getAllSprints
  - getIssuesForSprint
  - getAllVersions
</details>
<details>
  <summary>comment (/rest/api/2/comment)</summary>

  - getCommentPropertyKeys
  - setCommentProperty
  - getCommentProperty
  - deleteCommentProperty
</details>
<details>
  <summary>component (/rest/api/2/component)</summary>

  - getComponent
  - createComponent
  - editComponent
  - deleteComponent
  - getRelatedIssueCount
</details>
<details>
  <summary>customFieldOption (/rest/api/2/customFieldOptions)</summary>

  - getCustomFieldOption
</details>
<details>
  <summary>epic (/rest/agile/1.0/epic)</summary>

  - getIssuesWithoutEpic
  - removeIssuesFromEpic
  - getEpic
  - partiallyUpdateEpic
  - getIssuesForEpic
  - moveIssuesToEpic
  - rankEpics
</details>
<details>
  <summary>dashboard (/rest/api/2/dashboard)</summary>

  - getAllDashboards
  - getDashboard
</details>
<details>
  <summary>developmentInformation (/rest/devinfo)</summary>

  - store
  - getRepository
  - deleteRepository
  - deleteByProperties
  - checkExists
  - deleteEntity
</details>
<details>
  <summary>field (/rest/api/2/field)</summary>

  - getAllFields
  - createCustomField
</details>
<details>
  <summary>filter (/rest/api/2/filter)</summary>

  - createFilter
  - getFilter
  - updateFilter
  - deleteFilter
  - getFilterColumns
  - setFilterColumns
  - resetFilterColumns
  - getDefaultShareScope
  - setDefaultShareScope
  - getFavoriteFilters
</details>
<details>
  <summary>group (/rest/api/2/group)</summary>

  - createGroup
  - getGroup [DEPRECATED, use getMembers]
  - addUserToGroup
  - removeUserFromGroup
  - deleteGroup
  - getMembers
</details>
<details>
  <summary>groups (/rest/api/2/groups)</summary>

  - findGroups
</details>
<details>
  <summary>groupUserPicker (/rest/api/2/groupuserpicker)</summary>

  - findUsersAndGroups
</details>
<details>
  <summary>issue (/rest/api/2/issue and /rest/agile/1.0/issue)</summary>

  - getIssue (agile api, set `opts.agile: true`)
  - getIssueEstimation (agile api)
  - setIssueEstimation (agile api)
  - setIssueRanks (agile api)
  - createIssue
  - getCreateMetadata
  - bulkCreate
  - getIssue
  - deleteIssue
  - editIssue
  - assignIssue
  - getComments
  - addComment
  - getComment
  - editComment
  - deleteComment
  - getEditMetadata
  - sendEmailNotification
  - getRemoteLinks
  - createRemoteLink
  - updateRemoteLink
  - deleteRemoteLink
  - getRemoteLinkByID
  - updateRemoteLinkByID
  - deleteRemoteLinkByID
  - getTransitions
  - transitionIssue
  - unvote
  - vote
  - getVotes
  - getWatchers
  - addWatcher
  - removeWatcher
  - getWorkLogs
  - addWorkLog
  - updateWorkLog
  - deleteWorkLog
  - addAttachment
  - getProperties
  - setProperty
  - getProperty
  - deleteProperty
  - getIssuePicker
  - getChangelog
</details>
<details>
  <summary>issueLink (/rest/api/2/issueLink)</summary>

  - createIssueLink
  - getIssueLink
  - deleteIssueLink
</details>
<details>
  <summary>issueLinkType (/rest/api/2/issueLinkType)</summary>

  - getAvailableTypes
  - createIssueLinkType
  - getIssueLinkType
  - deleteIssueLinkType
  - editIssueLinkType
</details>
<details>
  <summary>issueType (/rest/api/2/issuetype)</summary>

  - getAllIssueTypes
  - getIssueType
  - createIssueType
  - deleteIssueType
  - updateIssueType
  - getAlternativeIssueTypes
</details>
<details>
  <summary>jql (/rest/api/2/jql/autocompletedata)</summary>

  - getAutoCompleteData
</details>
<details>
  <summary>labels (/rest/api/1.0/labels/suggest?query= and /rest/api/2/label)</summary>

  - getLabels
  - getAllLabels
</details>
<details>
  <summary>licenseRole (/rest/api/2/licenserole)</summary>

  - getAllLicenseRoles
  - getLicenseRole
  - editLicenseRole
</details>
<details>
  <summary>licenseValidator (/rest/api/2/licenseValidator)</summary>

  - validateLicense
</details>
<details>
  <summary>myPermissions (/rest/api/2/mypermissions)</summary>

  - getMyPermissions
</details>
<details>
  <summary>myPreferences (/rest/api/2/mypreferences)</summary>

  - getPreference
  - editPreference
  - deletePreference
</details>
<details>
  <summary>myself (/rest/api/2/myself)</summary>

  - getMyself
  - editMyslef
  - changePassword
</details>
<details>
  <summary>password (/rest/api/2/password)</summary>

  - getPasswordPolicy
</details>
<details>
  <summary>permissions (/rest/api/2/permissions)</summary>

  - getAllPermissions
</details>
<details>
  <summary>permissionScheme (/rest/api/2/permissionscheme)</summary>

  - getAllPermissionSchemes
  - createPermissionScheme
  - getPermissionScheme
  - editPermissionScheme
  - deletePermissionScheme
  - getPermissionSchemeGrants
  - createPermissionGrantInScheme
  - deletePermissionGrantFromScheme
  - getPermissionSchemeGrantById
</details>
<details>
  <summary>priority (/rest/api/2/priority)</summary>

  - getAllPriorities
  - getPriority
</details>
<details>
  <summary>project (/rest/api/2/project)</summary>

  - getAllProjects
  - getProject
  - getComponents
  - getStatuses
  - getVersions
  - getVersionsPaginated
  - getRoles
  - getRole
  - updateRole
  - updateProject
  - addToRole
</details>
<details>
  <summary>projectCategory (/rest/api/2/projectCategory)</summary>

  - getAllProjectCategories
  - getProjectCategory
</details>
<details>
  <summary>projectValidate (/rest/api/2/projectvalidate)</summary>

  - validateProjectKey
</details>
<details>
  <summary>reindex (/rest/api/2/reindex)</summary>

  - doReindex
  - getReindex
</details>
<details>
  <summary>resolution (/rest/api/2/resolution)</summary>

  - getAllResolutions
  - getResolution
</details>
<details>
  <summary>roles (/rest/api/2/role) (a.k.a. ProjectRoles)</summary>

  - getAll
  - createRole
  - getRoleById
  - updateRole
  - deleteRole
  - getActors
  - addActors
  - removeActor
</details>
<details>
  <summary>screens (/rest/api/2/screens)</summary>

  - getAvailableFields
  - getTabs
  - createTab
  - renameTab
  - deleteTab
  - addFieldToTab
  - getFieldsInTab
  - removeFieldFromTab
  - moveFieldOnTab
  - moveTabPosition
  - addFieldToDefaultTab
</details>
<details>
  <summary>search (/rest/api/2/search)</summary>

  - search
</details>
<details>
  <summary>securityLevel (/rest/api/2/securitylevel)</summary>

  - getSecurityLevel
</details>
<details>
  <summary>serverInfo (/rest/api/2/serverInfo)</summary>

  - getServerInfo
</details>
<details>
  <summary>settings (/rest/api/2/settings)</summary>

  - setBaseUrl
  - getIssueNavColumns
</details>
<details>
  <summary>sprint (/rest/agile/1.0/sprint)</summary>

  - createSprint
  - getSprint
  - updateSprint
  - partiallyUpdateSprint
  - deleteSprint
  - getSprintIssues
  - moveSprintIssues
  - swapSprint
</details>
<details>
  <summary>status (/rest/api/2/status)</summary>

  - getAllStatuses
  - getStatus
</details>
<details>
  <summary>statusCategory (/rest/api/2/statuscategory)</summary>

  - getAllStatusCategories
  - getStatusCategory
</details>
<details>
  <summary>user (/rest/api/2/user)</summary>

  - getUser
  - deleteUser
  - editUser
  - multiProjectSearchAssignable
  - searchAssignable
  - createTemporaryAvatar
  - convertTemporaryAvatar
  - deleteAvatar
  - getAvatars
  - getDefaultColumns
  - setDefaultColumns
  - resetDefaultColumns
  - changePassword
  - searchPermissions
  - searchPicker
  - search
  - all
  - viewIssueSearch
</details>
<details>
  <summary>version (/rest/api/2/version)</summary>

  - createVersion
  - moveVersion
  - getVersion
  - editVersion
  - getRelatedIssueCounts
  - getUnresolvedIssueCount
  - getRemoteLinks
  - createRemoteLink
  - getRemoteLink
  - deleteRemoteLink
  - deleteVersion
  - deleteAllRemoteLinks
  - getGlobalRemoteLink
</details>
<details>
  <summary>webhook (/rest/webhooks/1.0/webhook)</summary>

  - getAllWebhooks
  - getWebhook
  - createWebhook
  - deleteWebhook
</details>
<details>
  <summary>workflow (/rest/api/2/workflow)</summary>

  - getWorkflows
</details>
<details>
  <summary>workflowScheme (/rest/api/2/workflowscheme)</summary>

  - createWorkflowScheme
  - editWorkflowScheme
  - deleteWorkflowScheme
  - createDraft
  - getDefaultWorkflow
  - removeDefaultWorkflow
  - setDefaultWorkflow
  - getDraft
  - editDraft
  - deleteDraft
  - getDraftDefaultWorkflow
  - setDraftDefaultWorkflow
  - removeDraftDefaultWorkflow
  - getIssueType
  - removeIssueType
  - editIssueType
  - getDraftIssueType
  - editDraftIssueType
  - removeDraftIssueType
  - getWorkflow
  - editWorkflow
  - deleteWorkflow
  - getDraftWorkflow
  - editDraftWorkflow
  - deleteDraftWorkflow
</details>
<details>
  <summary>worklog (/rest/api/2/worklog)</summary>

  - getWorklogDeleted
  - worklogList
  - getWorklogUpdated
</details>

## License

[MIT license](http://opensource.org/licenses/MIT)
