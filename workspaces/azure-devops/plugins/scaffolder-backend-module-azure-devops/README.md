# Azure DevOps Actions backend module for the Scaffolder plugin

This is a [Backend Plugin](https://backstage.io/docs/plugins/backend-plugin/) containing a set of [Custom Actions](https://backstage.io/docs/features/software-templates/writing-custom-actions) using the Azure Devops [`REST API`](https://learn.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-7.2) & [azure-devops-node-api](https://github.com/microsoft/azure-devops-node-api).
It contains the following actions:

- `azure:pipeline:run`: run a azure pipeline for a project, with pipeline id
- `azure:pipeline:create`: Create a new Azure pipeline in a project.
- `azure:pipeline:permit`: Permit or deny a pipeline for a resource.
- `azure:repository:clone`: Clone an Azure repository for the context of the scaffolder
- `azure:repository:push`: Push changes back to Azure repository
- `azure:pr:create`: Create a new Pull Request

## Installation

From your Backstage instance root folder:

```shell
yarn --cwd packages/backend add @backstage-community/plugin-scaffolder-backend-module-azure-devops
```

## Configuration

### Code

Import the module by modifying the `packages/backend/src/index.ts` file on your Backstage instance:

```ts
// packages/backend/src/index.ts

backend.add(
  import('@backstage-community/plugin-scaffolder-backend-module-azure-devops'),
);
```

### Pre-Requisites

Your Backstage instance must have a valid [Azure DevOps integration](https://backstage.io/docs/integrations/azure/locations) for for your target host. `dev.azure.com` is used by default in the scaffolder actions.

To get started quickly, use the following as an example:

```yaml
integrations:
  azure:
    - host: dev.azure.com
      credentials:
        - personalAccessToken: ${PERSONAL_ACCESS_TOKEN}
```

To use other authentication methods, see the [documentation.](https://backstage.io/docs/integrations/azure/locations)

## Usage

You can use the action in any of the steps of your [Software Template](https://backstage.io/docs/features/software-templates/).
See [here](../../examples/scaffolder.yaml) for an example of a Software Template making use of the Azure Run Action.

---

## Actions Reference

### azure:pipeline:create

Creates a pipeline definition in Azure DevOps from a YAML file.

#### Input Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `organization` | `string` | ✅ | - | The name of the Azure DevOps organization |
| `project` | `string` | ✅ | - | The name of the Azure project |
| `repository` | `string` | ✅ | - | The repository name for the pipeline |
| `pipelineName` | `string` | ✅ | - | The name of the pipeline to create |
| `pipelineYamlFile` | `string` | ✅ | - | The path to the pipeline YAML file |
| `host` | `string` | ❌ | `dev.azure.com` | The Azure DevOps host |
| `branch` | `string` | ❌ | `main` | The branch of the pipeline's repository |
| `pipelineFolder` | `string` | ❌ | `/` | The folder where to create the pipeline |
| `pipelineAgentPoolName` | `string` | ❌ | `Azure Pipelines` | The name of the agent pool |
| `token` | `string` | ❌ | - | Token for Azure DevOps REST API authentication |

#### Output Values

| Parameter | Type | Description |
|-----------|------|-------------|
| `pipelineUrl` | `string` | URL of the created pipeline |
| `pipelineId` | `string` | The pipeline ID |

---

### azure:pipeline:run

Runs an existing Azure DevOps pipeline.

#### Input Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `organization` | `string` | ✅ | - | The name of the Azure DevOps organization |
| `project` | `string` | ✅ | - | The name of the Azure project |
| `pipelineId` | `string` | ✅ | - | The pipeline ID to execute |
| `host` | `string` | ❌ | `dev.azure.com` | The Azure DevOps host |
| `branch` | `string` | ❌ | `main` | The branch of the pipeline's repository |
| `token` | `string` | ❌ | - | Token for Azure DevOps REST API authentication |
| `pollingInterval` | `number` | ❌ | `0` | Seconds between each poll for pipeline status. `0` = no polling |
| `pipelineTimeout` | `number` | ❌ | - | Max seconds to wait for pipeline completion. Only effective if `pollingInterval` > 0 |
| `templateParameters` | `object` | ❌ | - | Azure DevOps pipeline template parameters in key-value pairs |
| `failIfNotSuccessful` | `boolean` | ❌ | `false` | Fail the action if the pipeline run was not successful |

#### Output Values

| Parameter | Type | Description |
|-----------|------|-------------|
| `pipelineRunUrl` | `string` | URL of the pipeline run |
| `pipelineRunStatus` | `enum` | Pipeline run status: `canceled`, `failed`, `succeeded`, `unknown` |
| `pipelineRunId` | `number` | The pipeline run ID |
| `pipelineTimeoutExceeded` | `boolean` | `true` if the pipeline did not complete within the defined timespan |
| `pipelineOutput` | `object` | Object containing output variables from the pipeline |

---

### azure:pipeline:permit

Authorizes or revokes a pipeline's access to an Azure DevOps resource.

#### Input Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `organization` | `string` | ✅ | - | The name of the Azure DevOps organization |
| `project` | `string` | ✅ | - | The name of the Azure project |
| `pipelineId` | `string` | ✅ | - | The pipeline ID |
| `resourceId` | `string` | ✅ | - | The resource ID |
| `resourceType` | `string` | ✅ | - | The resource type: `endpoint`, `repository`, `variablegroup`, `queue`, `securefile` |
| `host` | `string` | ❌ | `dev.azure.com` | The Azure DevOps host |
| `authorized` | `boolean` | ❌ | `true` | Authorize or revoke access |
| `apiVersion` | `string` | ❌ | `7.1-preview.1` | API version for authorization |
| `token` | `string` | ❌ | - | Token for Azure DevOps REST API authentication |

#### Output Values

This action does not produce any output.

#### Supported Resource Types

| Resource Type | Description |
|---------------|-------------|
| `endpoint` | Service Connections |
| `repository` | Git Repositories |
| `variablegroup` | Variable Groups |
| `queue` | Agent Queues |
| `securefile` | Secure Files |

---

### azure:repository:clone

Clones an Azure DevOps repository into the scaffolder workspace directory.

#### Input Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `remoteUrl` | `string` | ✅ | - | The Git URL to the repository |
| `branch` | `string` | ❌ | `main` | The branch to checkout |
| `targetPath` | `string` | ❌ | `./` | The subdirectory of the workspace to clone the repository into |
| `cloneDepth` | `number` | ❌ | - | Performs a shallow fetch, retrieving only the latest n commits |
| `token` | `string` | ❌ | - | Token for authentication |

#### Output Values

| Parameter | Type | Description |
|-----------|------|-------------|
| `cloneFullPath` | `string` | The full directory path where the repository was cloned |

---

### azure:repository:push

Pushes the workspace content to a remote Azure DevOps repository.

#### Input Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `remoteUrl` | `string` | ✅ | - | The Git URL to the repository |
| `branch` | `string` | ❌ | `scaffolder` | The branch to push to |
| `sourcePath` | `string` | ❌ | `.` | The subdirectory of the workspace containing the repository |
| `gitCommitMessage` | `string` | ❌ | `Initial commit` | The commit message |
| `gitAuthorName` | `string` | ❌ | - | The commit author name |
| `gitAuthorEmail` | `string` | ❌ | - | The commit author email |
| `token` | `string` | ❌ | - | Token for authentication |

#### Output Values

This action does not produce any output.

---

### azure:pr:create

Creates a Pull Request in an Azure DevOps repository.

#### Input Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `title` | `string` | ✅ | - | The Pull Request title |
| `repoName` | `string` | ✅ | - | The repository name |
| `project` | `string` | ✅ | - | The Azure DevOps project |
| `organization` | `string` | ❌ | - | The Azure DevOps organization name |
| `sourceBranch` | `string` | ❌ | `scaffolder` | The source branch to merge |
| `targetBranch` | `string` | ❌ | `main` | The target branch |
| `description` | `string` | ❌ | - | The Pull Request description |
| `supportsIterations` | `boolean` | ❌ | - | Whether the PR supports iterations |
| `server` | `string` | ❌ | `dev.azure.com` | The Azure DevOps service hostname |
| `token` | `string` | ❌ | - | Token for authentication |
| `autoComplete` | `boolean` | ❌ | `false` | Enable auto-completion once policies are met |
| `workItemId` | `string` | ❌ | - | The work item ID to associate with the PR |

#### Output Values

| Parameter | Type | Description |
|-----------|------|-------------|
| `pullRequestId` | `number` | The ID of the created Pull Request |

---

## Examples

### Example with the `azure:pipeline:run` action

```yaml
spec:
  parameters:
    - title: Choose a location
      required:
        - repoUrl
        - pipelineId
      properties:
        repoUrl:
          title: Repository Location
          type: string
          ui:field: RepoUrlPicker
          ui:options:
            allowedHosts:
              - dev.azure.com
        pipelineId:
          title: Pipeline Id
          type: string
          description: The Id for the pipeline you want to execute
  steps:
    - id: runAzurePipeline
      name: Run Pipeline
      action: azure:pipeline:run
      input:
        organization: ${{ (parameters.repoUrl | parseRepoUrl)['organization'] }}
        pipelineId: ${{ parameters.pipelineId }}
        project: ${{ (parameters.repoUrl | parseRepoUrl)['project'] }}

  output:
    links:
      - title: Open in DevOps
        icon: catalog
        url: ${{ steps['runAzurePipeline'].output.pipelineRunUrl }}
```

### Example Run Azure Pipeline on alternative host to dev.azure.com

```yaml
spec:
  #[...]
  steps:
    - id: runAzurePipeline
      name: Run Pipeline
      action: azure:pipeline:run
      input:
        #[...]
        host: 'AZURE_DEVOPS_HOST'
```

### Example using the User Token using `azure:pipeline:run` action

```yaml
spec:
  parameters:
    #[...]
    properties:
      repoUrl:
        ui:field: RepoUrlPicker
        ui:options:
          #[...]
          requestUserCredentials:
            secretsKey: USER_OAUTH_TOKEN
            additionalScopes:
              azure: []
      pipelineId:
        title: Pipeline Id
        type: string
        description: The Id for the pipeline you want to execute

  steps:
    - id: runAzurePipeline
      name: Run Pipeline
      action: azure:pipeline:run
      input:
        #[...]
        pipelineId: ${{ parameters.pipelineId }}
        token: ${{ secrets.USER_OAUTH_TOKEN }}
```

### Example running a pipeline on a branch using the `azure:pipeline:run` action

```yaml
spec:
  parameters:
    #[...]
    properties:
      #[...]
      branch:
        title: branch
        type: string
        description: The branch for the pipeline you want to execute

  steps:
    - id: runAzurePipeline
      name: Run Pipeline
      action: azure:pipeline:run
      input:
        #[...]
        branch: ${{ parameters.branch }}
```

### Example running a pipeline with template parameters using the `azure:pipeline:run` action

```yaml
spec:
  parameters:
    #[...]
    properties:
      #[...]
      templateParameters:
        type: object
        title: Template Parameters
        description: Azure DevOps pipeline template parameters in key-value pairs.

  steps:
    - id: runAzurePipeline
      name: Run Pipeline
      action: azure:pipeline:run
      input:
        #[...]
        templateParameters:
          projectRepo: ${{ (parameters.repoUrl | parseRepoUrl)['repo'] }}
          sampleTemplateParameterKey: sampleTemplateParameterValue
```

### Example running a pipeline, waiting for it to complete, and printing the output using the `azure:pipeline:run` action

```yaml
spec:
  steps:
    - id: runAzurePipeline
      name: Run Pipeline
      action: azure:pipeline:run
      input:
        #[...]
        pollingInterval: 10 # Poll for pipeline run status every 10 second
        pipelineTimeout: 300 # Timeout after 5 minutes
  output:
    text:
      - title: Pipeline run info
        content: |
          **pipelineRunStatus:** `${{ steps['runAzurePipeline'].output.pipelineRunStatus }}` }}
          **pipelineRunId:** `${{ steps['runAzurePipeline'].output.pipelineRunId }}` }}
          **pipeline output:** `${{ steps['runAzurePipeline'].output.pipelineOutput['myOutputVar'].value }}` }}
```

### Example running a pipeline, waiting for it to complete, and failing the action if the pipeline is unsuccessful using the `azure:pipeline:run` action

```yaml
spec:
  steps:
    - id: runAzurePipeline
      name: Run Pipeline
      action: azure:pipeline:run
      input:
        #[...]
        pollingInterval: 10 # Poll for pipeline run status every 10 seconds
        pipelineTimeout: 300 # Timeout after 5 minutes
        failIfNotSuccessful: true # Fail the scaffolder step when the run is not successful
```

### Example: Create Azure Pipeline

```yaml
spec:
  steps:
    - id: createAzurePipeline
      name: Create Pipeline
      action: azure:pipeline:create
      input:
        organization: ${{ (parameters.repoUrl | parseRepoUrl)['organization'] }}
        project: ${{ (parameters.repoUrl | parseRepoUrl)['project'] }}
        repository: ${{ (parameters.repoUrl | parseRepoUrl)['repo'] }}
        pipelineName: MyNewPipeline
        pipelineYamlFile: azure-pipelines.yml
        branch: main
  output:
    links:
      - title: Open Pipeline
        url: ${{ steps['createAzurePipeline'].output.pipelineUrl }}
```

### Example: Create Azure Pipeline on alternative host to dev.azure.com

```yaml
spec:
  steps:
    - id: createAzurePipeline
      name: create Azure Devops Pipeline
      action: azure:pipeline:create
      input:
        host: azure.com
        organization: org
        repository: repo
        pipelineName: pipeline
        pipelineYamlFile: pipeline.yaml
        project: project
  output:
    links:
      - title: Open Pipeline
        url: ${{ steps['createAzurePipeline'].output.pipelineUrl }}
```

### Example: Create Azure Pipeline from a target branch

```yaml
spec:
  steps:
    - id: createAzurePipeline
      name: Create Azure Devops Pipeline
      action: azure:pipeline:create
      input:
        organization: org
        repository: repo
        pipelineName: pipeline
        pipelineYamlFile: pipeline.yaml
        project: project
        branch: branch
```

### Example: Create Azure Pipeline inside a folder

```yaml
spec:
  steps:
    - id: createAzurePipeline
      name: Create Azure Devops Pipeline
      action: azure:pipeline:create
      input:
        organization: org
        repository: repo
        pipelineName: pipeline
        pipelineYamlFile: pipeline.yaml
        project: project
        pipelineFolder: folder
```

### Example: Create Azure Pipeline with custom token

```yaml
spec:
  parameters:
    #[...]
    properties:
      repoUrl:
        ui:field: RepoUrlPicker
        ui:options:
          #[...]
          requestUserCredentials:
            secretsKey: USER_OAUTH_TOKEN
            additionalScopes:
              azure: []
      pipelineId:
        title: Pipeline Id
        type: string
        description: The Id for the pipeline you want to execute
  steps:
    - id: createAzurePipeline
      name: Create Azure Devops Pipeline (custom token)
      action: azure:pipeline:create
      input:
        #[...]
        token: ${{ secrets.USER_OAUTH_TOKEN }}
```

### Example: Create Azure Pipeline with custom agent pool

```yaml
spec:
  steps:
    - id: createAzurePipeline
      name: Create Azure Devops Pipeline (custom agent pool)
      action: azure:pipeline:create
      input:
        organization: org
        repository: repo
        pipelineName: pipeline
        pipelineYamlFile: pipeline.yaml
        project: project
        pipelineAgentPoolName: my-agent-pool
```

### Example: Permit Azure Pipeline

```yaml
spec:
  steps:
    - id: permitAzurePipeline
      name: Permit Pipeline
      action: azure:pipeline:permit
      input:
        organization: ${{ (parameters.repoUrl | parseRepoUrl)['organization'] }}
        project: ${{ (parameters.repoUrl | parseRepoUrl)['project'] }}
        pipelineId: ${{ parameters.pipelineId }}
        resourceId: ${{ parameters.resourceId }}
        resourceType: endpoint
        authorized: true
```

> [!WARNING]  
> The `azure:pipeline:permit` action uses the Azure DevOps REST API version `7.1-preview.1` by default.  
> You can override this by setting the `apiVersion` input parameter as shown above.  
> The action only supports API version `7.1-preview.1` or newer; older versions are not compatible.

### Example: Pipeline Creation and Permit Azure Pipeline

```yaml
spec:
  steps:
    - id: createAzurePipeline
      action: azure:pipeline:create
      name: Create Azure Devops Pipeline
      input:
        organization: org
        repository: repo
        pipelineName: pipeline
        pipelineYamlFile: pipeline.yaml
        project: project
    - id: permitPipelineResources
      action: azure:pipeline:permit
      name: Authorize Pipeline Resources
      input:
        organization: org
        project: project
        pipelineId: ${{ steps.createPipeline.output.pipelineId }}
        resourceId: ${{ parameters.serviceConnectionId }}
        resourceType: endpoint
        authorized: true
```

### Example: Multiple Resource Authorization for Single Pipeline

```yaml
spec:
  steps:
    - id: permitServiceConnection
      action: azure:pipeline:permit
      name: Authorize Service Connection
      input:
        organization: org
        project: project
        pipelineId: '777'
        resourceId: service-connection-id
        resourceType: endpoint
        authorized: true
    - id: permitSharedRepo
      action: azure:pipeline:permit
      name: Authorize Shared Repository
      input:
        organization: org
        project: project
        pipelineId: '777'
        resourceId: shared-templates-repo
        resourceType: repository
        authorized: true
    - id: permitVariables
      action: azure:pipeline:permit
      name: Authorize Variable Group
      input:
        organization: org
        project: project
        pipelineId: '777'
        resourceId: '15'
        resourceType: variablegroup
        authorized: true
```

### Example: Clone an Azure Git Repository

```yaml
spec:
  steps:
    - id: cloneAzureRepo
      name: git clone
      action: azure:repository:clone
      input:
        remoteUrl: 'https://dev.azure.com/{organization}/{project}/_git/{repository}'
        branch: 'main'
        targetPath: ./work
        token: ${{ secrets.USER_OAUTH_TOKEN }}
```

### Example: Clone an Azure Git Repository With Shallow Depth

Creates a shallow clone with the history truncated to the specified number of commits, useful for reducing data transfer and clone time for large repositories.

```yaml
spec:
  steps:
    - id: cloneAzureRepo
      name: git clone
      action: azure:repository:clone
      input:
        remoteUrl: 'https://dev.azure.com/{organization}/{project}/_git/{repository}'
        branch: 'main'
        targetPath: ./work
        cloneDepth: 1
        token: ${{ secrets.USER_OAUTH_TOKEN }}
```

### Example: Push changes made back to Git Repository

```yaml
spec:
  steps:
    - id: pushAzureRepoRepos
      name: git push
      action: azure:repository:push
      input:
        branch: scaffolder/new-branch
        sourcePath: ./work
        gitCommitMessage: My changes
        token: ${{ secrets.USER_OAUTH_TOKEN }}
```

### Example: Create a new Simple Pull Request

```yaml
spec:
  steps:
    - id: createPR
      name: Create PR
      action: azure:pr:create
      input:
        organization: org
        repoName: repo
        title: My PR
        project: my-project
        description: 'Created by Backstage Scaffolder'
        token: ${{ secrets.USER_OAUTH_TOKEN }}
```

### Example: Create a new Pull Request

```yaml
spec:
  steps:
    - id: createPR
      name: Create PR
      action: azure:pr:create
      input:
        organization: org
        sourceBranch: scaffolder/new-branch
        targetBranch: 'main'
        repoName: repo
        title: My PR
        project: my-project
        supportsIterations: true
        description: 'Created by Backstage Scaffolder'
        autoComplete: true
        workItemId: 12345
        token: ${{ secrets.USER_OAUTH_TOKEN }}
```
