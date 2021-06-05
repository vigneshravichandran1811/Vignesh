# Lab: Configuring Agent Pools and Understanding Pipeline Styles
## Objectives

After you complete this lab, you will be able to:

- implement YAML-based pipelines

## Lab duration

-   Estimated time: **30 minutes**

## Instructions

### Before you start

#### Set up an Azure DevOps organization

If you don't already have an Azure DevOps organization that you can use for this lab, create one by following the instructions available at [Create an organization or project collection](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

### Exercise 0: Configure the lab prerequisites

In this exercise, you will set up the prerequisite for the lab, which consists of the preconfigured Parts Unlimited team project based on an Azure DevOps Demo Generator template.

#### Task 1: Configure the team project

In this task, you will use Azure DevOps Demo Generator to generate a new project based on the **PartsUnlimited** template.

1.  On your lab computer, start a web browser and navigate to [Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net). This utility site will automate the process of creating a new Azure DevOps project within your account that is prepopulated with content (work items, repos, etc.) required for the lab.

    > **Note**: For more information on the site, see https://docs.microsoft.com/en-us/azure/devops/demo-gen.

1.  Click **Sign in** and sign in using the Microsoft account associated with your Azure DevOps subscription.
1.  If required, on the **Azure DevOps Demo Generator** page, click **Accept** to accept the permission requests for accessing your Azure DevOps subscription.
1.  On the **Create New Project** page, in the **New Project Name** textbox, type **Configuring Agent Pools and Understanding Pipeline Styles**, in the **Select organization** dropdown list, select your Azure DevOps organization, and then click **Choose template**.
1.  On the **Choose a template** page, click the **PartsUnlimited** template, and then click **Select Template**.
1.  Click **Create Project**

    > **Note**: Wait for the process to complete. This should take about 2 minutes. In case the process fails, navigate to your DevOps organization, delete the project, and try again.

1.  On the **Create New Project** page, click **Navigate to project**.

### Exercise 1: Author YAML-based Azure DevOps pipelines

In this exercise, you will convert a classic Azure DevOps pipeline into a YAML-based one.

#### Task 1: Create an Azure DevOps YAML pipeline

In this task, you will create a template-based Azure DevOps YAML pipeline.

1.  From the web browser displaying the Azure DevOps portal with the **Configuring Agent Pools and Understanding Pipeline Styles** project open, in the vertical navigational pane on the left side, click **Pipelines**.
1.  On the **Recent** tab of the **Pipelines** pane, click **New pipeline**.
1.  On the **Where is your code?** pane, click **Azure Repos Git**.
1.  On the **Select a repository** pane, click **PartsUnlimited**.
1.  On the **Configure your pipeline** pane, click **Starter pipeline**.
1.  On the **Review your pipeline YAML** pane, review the sample pipeline, click the down-facing caret symbol next to the **Save and run** button, click **Save** and, on the **Save** pane, click **Save**.

    > **Note**: This will result in creation of the **azure-pipelines.yml** file in the root directory of the repository hosting the project code.

    > **Note**: You will replace the content of the sample YAML pipeline with the code of pipeline generated by the classic editor and modify it to account for differences between the classic and YAML pipelines.

#### Task 2: Convert a classic pipeline into a YAML pipeline

In this task, you will convert a classic pipeline into a YAML pipeline

1.  From the web browser displaying the Azure DevOps portal with the **Configuring Agent Pools and Understanding Pipeline Styles** project open, in the vertical navigational pane on the left side, click **Pipelines**.
1.  On the **Recent** tab of the **Pipelines** pane, hover with the mouse pointer over the right edge of the entry containing the **PartsUnlimitedE2E** entry to reveal the vertical ellipsis symbol designating the **More** menu, click the ellipsis symbol, and, in the dropdown menu, click **Edit**. This will display the build pipeline that is part of the project you generated at the beginning of the lab.
1.  On the **Tasks** tab of the **PartsUnlimitedE2E** edit pane, click **Triggers**, on the right side of the **PartsUnlimited** pane, uncheck the **Enable continuous integration** checkbox, click the down-facing caret next to the **Save & queue** button, in the dropdown menu, click **Save**, and in the **Save build pipeline** click **Save**.

    > **Note**: This will prevent from unintended execution of automatic build due to changes to the repository during this lab.

    > **Note**: Alternatively, you could simply delete the existing pipeline once you copy its content into the new one.

1.  In the Azure DevOps portal, in the vertical navigational pane on the left side, in the **Pipelines** section, click **Pipelines**.
1.  On the **Recent** tab of the **Pipelines** pane, click the **PartsUnlimitedE2E** entry.
1.  On the **Runs** tab of the **PartsUnlimitedE2E** pane, in the upper right corner, click the vertical ellipsis (three vertical dots) symbol and, in the dropdown menu, click **Export to YAML**. This will automatically download the **PartsUnlimitedE2E.yml** file to your local **Downloads** folder.

    > **Note**: The **Export to YAML** feature replaces an older **View YAML** option available from the pipeline editor pane within the Azure DevOps portal, which was limited to viewing YAML content one job at a time. The new functionality leverages existing classic and YAML pipeline infrastructure, including YAML parsing library, which results in more accurate translation between the two. It supports the following pipeline components:

    - Single and multiple jobs
    - Checkout options
    - Execution plan parallelism
    - Timeout and name metadata
    - Demands
    - Schedules and other triggers
    - Pool selection, including jobs which differ from the default
    - All tasks and all inputs, including optimizing for default inputs
    - Job and step conditions
    - Task group unrolling

    > **Note**: The only components not covered by the new functionality are variables and timezone translation. Variables defined in YAML override variables set in the user interface of the Azure DevOps portal. If the **Export to YAML** feature detects presence of Classic pipeline variables, they will be explicitly included in the comments within the newly generated YAML pipeline definition. Similarly, since cron schedules in YAML are expressed in UTC, while classic schedules rely on the organization’s timezone, their presence is also included in the comments.

    > **Note**: For more information regarding this functionality, refer to [Replacing "View YAML"](https://devblogs.microsoft.com/devops/replacing-view-yaml/)

1.  On the lab computer, start Visual Studio Code and use it to open the file **PartsUnlimitedE2E.yml**. The file should have the following content:

    ```yaml
    name: $(date:yyyyMMdd)$(rev:.r)
    jobs:
    - job: Phase_1
      displayName: Phase 1
      cancelTimeoutInMinutes: 1
      pool:
        vmImage: vs2017-win2016
      steps:
      - checkout: self
      - task: NuGetInstaller@0
        name: NuGetInstaller_1
        displayName: NuGet restore
        inputs:
          solution: '**\*.sln'
      - task: VSBuild@1
        name: VSBuild_2
        displayName: Build solution
        inputs:
          vsVersion: 15.0
          msbuildArgs: /p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="$(build.stagingDirectory)" /p:IncludeServerNameInBuildInfo=True /p:GenerateBuildInfoConfigFile=true /p:BuildSymbolStorePath="$(SymbolPath)" /p:ReferencePath="C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\Extensions\Microsoft\Pex"
          platform: $(BuildPlatform)
          configuration: $(BuildConfiguration)
      - task: VSTest@1
        name: VSTest_3
        displayName: Test Assemblies
        inputs:
          testAssembly: '**\$(BuildConfiguration)\*test*.dll;-:**\obj\**'
          codeCoverageEnabled: true
          vsTestVersion: latest
          platform: $(BuildPlatform)
          configuration: $(BuildConfiguration)
      - task: CopyFiles@2
        name: CopyFiles1
        displayName: Copy Files
        inputs:
          SourceFolder: $(build.sourcesdirectory)
          Contents: '**/*.json'
          TargetFolder: $(build.artifactstagingdirectory)
      - task: PublishBuildArtifacts@1
        name: PublishBuildArtifacts_5
        displayName: Publish Artifact
        inputs:
          PathtoPublish: $(build.artifactstagingdirectory)
          TargetPath: '\\my\share\$(Build.DefinitionName)\$(Build.BuildNumber)'
    ...
    ```

1.  In the Visual Studio Code window, in the top-level menu, click **Selection** and, in the dropdown menu, click **Select All**.
1.  In the Visual Studio Code window, in the top-level menu, click **Edit** and, in the dropdown menu, click **Copy**.
1.  Switch to the browser window displaying the Azure DevOps portal and, in the vertical navigational pane on the left side, in the **Pipelines** section, click **Pipelines**.
1.  On the **Recent** tab of the **Pipelines** pane, select **PartsUnlimited** and, on the **PartsUnlimited** pane, select **Edit**.
1.  On the **PartsUnlimited** edit pane, select the existing YAML content of the pipeline, replace it with the content of Clipboard.
1.  On the **PartsUnlimited** edit pane, in the upper-right corner, click **Save**, and, on the **Save** pane, click **Save**. This will automatically trigger the build based on this pipeline.
1.  In the Azure DevOps portal, in the vertical navigational pane on the left side, in the **Pipelines** section, click **Pipelines**.
1.  On the **Recent** tab of the **Pipelines** pane, click the **PartsUnlimited** entry, on the **Runs** tab of the **PartsUnlimited** pane, select the most recent run, on the **Summary** pane of the run, scroll down to the bottom, in the **Jobs** section, click **Phase 1** and monitor the job until its successful completion.