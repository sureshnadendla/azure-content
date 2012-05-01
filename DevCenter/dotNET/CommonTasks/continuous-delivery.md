<properties linkid="dev-net-common-tasks-continuous-delivery" urlDisplayName="Continuous Delivery" headerExpose="" pageTitle="Continuous Delivery for Cloud Applications in Windows Azure" metaKeywords="" footerExpose="" metaDescription="" umbracoNaviHide="0" disqusComments="1" />
  <h1>Continuous Delivery for Cloud Applications in Windows Azure</h1>
  <p>The process described in this article shows you how to set up continuous delivery for Windows Azure cloud apps. This process enables you to automatically create packages and deploy the package to Windows Azure after every code check-in. The package build process described in this article is equivalent to the Package command in Visual Studio, and the publishing steps are equivalent to the Publish command in Visual Studio. The article covers the methods you would use to create a build server with MSBuild command-line statements and Windows PowerShell scripts, and it also demonstrates how to optionally configure Visual Studio Team Foundation Server - Team Build definitions to use the MSBuild commands and PowerShell scripts. The process is customizable for your build environment and Windows Azure target environments.</p>
  <p>This task includes the following steps:</p>
  <ul>
    <li>
      <a href="#step1">Step 1: Configure the Build Server</a>
    </li>
    <li>
      <a href="#step2">Step 2: Build a Package using MSBuild Commands</a>
    </li>
    <li>
      <a href="#step3">Step 3: Build a Package using TFS Team Build (Optional)</a>
    </li>
    <li>
      <a href="#step4">Step 4: Publish a Package using a PowerShell Script</a>
    </li>
    <li>
      <a href="#step5">Step 5: Publish a Package using TFS Team Build (Optional)</a>
    </li>
  </ul>
  <h2>
    <a name="step1">
    </a>Step 1: Configure the Build Server</h2>
  <p>Before you can create a Windows Azure package by using MSBuild, you must install the required software and tools on the build server.</p>
  <p>Visual Studio is not required to be installed on the build server. If you want to use Team Foundation Build Service to manage your build server, follow the instructions in the <a href="http://go.microsoft.com/fwlink/p/?LinkId=239963">Team Foundation Build Service</a> documentation.</p>
  <ol>
    <li>On the build server, install the <a href="http://go.microsoft.com/fwlink/?LinkId=239538">.NET Framework 4</a>, which includes MSBuild.</li>
    <li>Install the <a href="http://go.microsoft.com/fwlink/?LinkId=239600">Windows Azure Authoring Tools</a> (look for WindowsAzureSDK-x86.msi or WindowsAzureSDK-x64.msi, depending on your build server's processor).</li>
    <li>Copy the Microsoft.WebApplication.targets file from a Visual Studio installation to the build server.On a computer with Visual Studio installed, the file is located in the directory C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v10.0\WebApplications. You should copy it to the same directory on the build server.</li>
    <li>Install the <a href="http://go.microsoft.com/fwlink/?LinkId=239600">Windows Azure Tools for Visual Studio</a>. Look for WindowsAzureTools.VS100.exe in the download list.</li>
  </ol>
  <h2>
    <a name="step2">
    </a>Step 2: Build a Package using MSBuild Commands</h2>
  <p>This section describes how to construct an MSBuild command that builds a Windows Azure package. Run this step on the build server to verify that everything is configured correctly and that the MSBuild command does what you want it to do. You can either add this command line to existing build scripts on the build server, or you can use the command line in a TFS Build Definition, as described in the next section. For more information about command-line parameters and MSBuild, see <a href="http://msdn.microsoft.com/en-us/library/ms164311(v=VS.90).aspx">MSBuild Command Line Reference</a>.</p>
  <ol>
    <li>
      <p>If Visual Studio 2010 is installed on the build server, click <strong>Start</strong>, click <strong>All Programs</strong>, and then locate and click <strong>Visual Studio Commmand Prompt</strong> in the <strong>Visual Studio Tools</strong> folder.</p>
      <p>If Visual Studio 2010 is not installed on the build server, open a command prompt and make sure that MSBuild.exe is accessible on the path. MSBuild is installed with the .NET Framework in the path <br />%WINDIR%\Microsoft.NET\Framework\<em>&lt;Version&gt;</em>. For example, to add MSBuild.exe to the PATH environment variable when you have .NET Framework 4 installed, type the following command at the command prompt:</p>
      <pre class="prettyprint">set PATH=%PATH%;"C:\Windows\Microsoft.NET\Framework\4.0"</pre>
    </li>
    <li>
      <p>At the command prompt, navigate to the folder containing the Windows Azure project file that you want to build.</p>
    </li>
    <li>
      <p>Run msbuild with the /target:Publish option as in the following example:</p>
      <pre class="prettyprint">MSBuild /target:Publish</pre>
      <p>This option can be abbreviated as /t:Publish. The /t:Publish option in MSBuild should not be confused with the Publish commands available in Visual Studio when you have the Windows Azure SDK installed. The /t:Publish option only builds the Windows Azure packages. It does not deploy the packages as the Publish commands in Visual Studio do.</p>
      <p>Optionally, you can specify the project name as an MSBuild parameter. If not specified, the current directory is used. For more information about MSBuild command line options, see <a href="http://go.microsoft.com/fwlink/p/?LinkId=239966">MSBuild Command Line Reference</a>.</p>
    </li>
    <li>
      <p>Locate the output. By default, this command creates a directory in relation to the root folder for the project, such as <em>&lt;ProjectDir&gt;</em>\bin\<em>&lt;Configuration&gt;</em>\app.publish\. When you build a Windows Azure project, you generate two files, the package file itself and the accompanying configuration file:</p>
      <ul>
        <li>Project.cspkg</li>
        <li>ServiceConfiguration.<em>&lt;TargetProfile&gt;</em>.cscfg</li>
      </ul>
      <p>By default, each Windows Azure project includes one service-configuration file (.cscfg file) for local (debugging) builds and another for cloud (staging or production) builds, but you can add or remove service-configuration files as needed. When you build a package within Visual Studio, you will be asked which service configuration file to include alongside the package.</p>
    </li>
    <li>
      <p>Specify the service configuration file. When you build a package by using MSBuild, the local service configuration file is included by default. To include a different service configuration file, set the TargetProfile property of the MSBuild command, as in the following example:</p>
      <pre class="prettyprint">MSBuild /t:Publish /p:TargetProfile=ServiceConfiguration.Cloud.cscfg</pre>
    </li>
    <li>
      <p>Specify the location for the output. Set the path by using the /p:PublishDir=<em>&lt;Directory&gt;</em>\ option, including the trailing backslash separator, as in the following example:</p>
      <pre class="prettyprint">MSBuild /target:Publish /p:PublishDir=\\myserver\drops\</pre>
      <p>Once you've constructed and tested an appropriate MSBuild command line to build your projects and combine them into an Azure package, you can add this command line to your build scripts. If your build server uses custom scripts, this process will depend on the specifics of your build custom process. If you are using TFS as a build environment, then you can follow the instructions in the next step to add the Windows Azure package build to your build process.</p>
    </li>
  </ol>
  <h2>
    <a name="step3">
    </a>Step 3: Build a Package using TFS Team Build (Optional)</h2>
  <p>If you have Team Foundation Server (TFS) set up as a build controller and the build server set up as a TFS build machine, then you can set up an automated build for your Windows Azure package. For information on how to set up and use Team Foundation server as a build system, see <a href="http://go.microsoft.com/fwlink/?LinkId=238798">Understanding the Team Foundation Build System</a>. In particular, the following procedure assumes that you have configured your build server as described in <a href="http://go.microsoft.com/fwlink/?LinkId=238799">Configure a Build Machine</a>.</p>
  <p>To configure TFS to build Windows Azure packages, perform the following steps:</p>
  <ol>
    <li>
      <p>In Visual Studio 2010 on your development computer, on the View menu, choose <strong>Team Explorer</strong>, or choose Ctrl+\, Ctrl+M. In the Team Explorer window, expand the <strong>Builds </strong>node, right-click <strong>All Build Definitions</strong>, and then click <strong>New Build Definition</strong>:</p>
      <p>
        <img src="../../../DevCenter/dotNet/Media/tfs-01.png" />
      </p>
    </li>
    <li>Click the <strong>Process </strong>tab. On the Process tab, choose the default template, and expand the <strong>Advanced </strong>section in the grid.</li>
    <li>
      <p>Choose <strong>MSBuild Arguments</strong>, and set the appropriate MSBuild command line arguments as described in Step 2 above. For example, enter <strong>/t:Publish /p:PublishDir=\\myserver\drops\ </strong>to build a package and copy the package files to the location \\myserver\drops\:</p>
      <p>
        <img src="../../../DevCenter/dotNet/Media/tfs-02.png" />
      </p>
      <p>
        <strong>Note:</strong> Copying the files to a public share makes it easier to manually deploy the packages from your development computer.</p>
    </li>
    <li>Click the <strong>Trigger </strong>tab, and specify the desired conditions for when you want the package to be built. For example, specify <strong>Continuous Integration</strong> to build the package whenever a source control check-in occurs.</li>
    <li>Test the success of your build step by checking in a change to your project, or queue up a new build. To queue up a new build, in the Team Explorer, right-click <strong>All Build Definitions,</strong> and then choose <strong>Queue New Build</strong>.</li>
  </ol>
  <h2>
    <a name="step4">
    </a>Step 4: Publish a Package using a Powershell Script</h2>
  <p>This section describes how to construct a Windows PowerShell script that will publish the Cloud app package output to Windows Azure using optional parameters. This script can be called after the build step in your custom build automation. It can also be called from Process Template workflow activities in Visual Studio TFS Team Build.</p>
  <ol>
    <li>
      <p>Install the <a href="http://go.microsoft.com/fwlink/?LinkId=242478">Windows Azure PowerShell Cmdlets</a> (v2.2.1 or higher). During the cmdlet setup phase choose to install as a snap-in.</p>
    </li>
    <li>
      <p>Start Windows PowerShell using the Start menu or typing PowerShell.exe at a command prompt.</p>
    </li>
    <li>
      <p>At the PowerShell prompt, initialize the PowerShell environment so that the Windows Azure Cmdlets snap-in is loaded</p>
      <pre class="prettyprint">Add-PSSnapin WAPPSCmdlets
Get-Command -Module WAPPSCmdlets</pre>
    </li>
    <li>
      <p>Install the certificates needed to allow the script to communicate with Windows Azure service management APIs. First download <a href="https://windows.azure.com/download/publishprofile.aspx?wa=wsignin1.0">the .publishsettings file</a> corresponding to your subscription – this file contains the certificate. Then install the certificate to the User Store and save the Subscription.xml file containing certificate thumbprint and subscription ID pair using the following cmdlet:</p>
      <pre class="prettyprint">Import-Subscription -PublishSettingsFile 'c:\downloads\mysubscription.publishsettings' -SubscriptionDataFile 'c:\scripts\WindowsAzure\Subscription.xml' </pre>
    </li>
    <li>
      <p>Edit the Subscription.xml file created in the prior step, and set the Name attribute of the Subscription element to "default". This makes it easier for all future steps to refer to this subscription by the name <strong>default</strong>. Next set the subscription described in Subscription.xml as an active subscription in memory using this command:</p>
      <pre class="prettyprint">Set-Subscription -SubscriptionDataFile 'c:\scripts\WindowsAzure\Subscription.xml'</pre>
    </li>
    <li>
      <p>Select the subscription named "default" for all subsequent Windows Azure cmdlet calls:</p>
      <pre class="prettyprint">Select-Subscription 'default'</pre>
    </li>
    <li>
      <p>Test the Windows Azure cmdlet against your subscription by running this command. The result should show you the set of Hosted Services in your subscription.</p>
      <pre class="printprint">Get-HostedService</pre>
    </li>
    <li>
      <p>Save the script template provided at the <a href="#script">end of this article</a> to your scripts folder as c:\scripts\WindowsAzure\<strong>PublishCloudApp.ps1</strong>.</p>
    </li>
    <li>
      <p>Review the parameters section of the script. Add or modify any default values. These values can always be overridden by passing in explicit parameters.</p>
    </li>
    <li>
      <p>Ensure there are valid hosted service and storage accounts created in your subscription that can be targeted by the publish script. The storage account (blob storage) will be used to upload and temporarily store the deployment package and config file while the deployment is being created.</p>
      <ul>
        <li>
          <p>To create a new hosted service, you can call this script or use the Windows Azure Management Portal. The hosted service name will be used as a prefix in a fully qualified domain name and hence it must be unique.</p>
          <pre class="prettyprint">New-HostedService -ServiceName "mytesthostedservice" -Location "North Central US" -Label "mytesthostedservice"</pre>
        </li>
        <li>
          <p>To create a new storage account, you can call this script or use the Windows Azure Management Portal. The storage account name will be used as a prefix in a fully qualified domain name and hence it must be unique. You can try using the same name as the hosted service.</p>
          <pre class="prettyprint">New-StorageAccount -ServiceName "mytesthostedservice" -Location "North Central US" -Label "mytesthostedservice"</pre>
        </li>
      </ul>
    </li>
    <li>
      <p>Call the script directly from Windows PowerShell, or wire up this script to your host build automation to occur after the package build.</p>
      <p>
        <strong>WARNING:</strong> The script will always delete or replace your existing deployments by default if they are detected. This is necessary to enable continuous delivery from automation where no user prompting is possible.</p>
      <p>
        <strong>Example scenario 1:</strong> continuous deployment to the staging environment of a service:</p>
      <pre class="prettyprint">PowerShell c:\scripts\windowsazure\PublishCloudApp.ps1 –environment Staging -serviceName myhostedservice -storageAccountName myhostedservice -packageLocation c:\drops\app.publish\ContactManager.Azure.cspkg -cloudConfigLocation c:\drops\app.publish\ServiceConfiguration.Cloud.cscfg</pre>
      <p>This is typically followed up by test run verification and a VIP swap. The VIP swap can be done via the Windows Azure Management Portal or by using the Move-Deployment cmdlet.</p>
      <p>
        <strong>Example scenario 2:</strong> continuous deployment to the production environment of a dedicated test service</p>
      <pre class="prettyprint">PowerShell c:\scripts\windowsazure\PublishCloudApp.ps1 –environment Production –enableDeploymentUpgrade 1 -serviceName myhostedservice -storageAccountName myhostedservice -packageLocation c:\drops\app.publish\ContactManager.Azure.cspkg -cloudConfigLocation c:\drops\app.publish\ServiceConfiguration.Cloud.cscfg</pre>
      <p>
        <strong>Remote Desktop:</strong>
      </p>
      <p>If Remote Desktop is enabled in your Windows Azure project you will need to perform additional one-time steps to ensure the correct Hosted Service Certificate is uploaded to all hosted services targeted by this script.</p>
      <p>Locate the certificate thumbprint values expected by your roles. The thumbprint values are visible in the Certificates section of the cloud config file (i.e. ServiceConfiguration.Cloud.cscfg). It is also visible in the Remote Desktop Configuration dialog in Visual Studio when you Show Options and view the selected certificate.</p>
      <pre class="prettyprint">&lt;Certificates&gt;
      &lt;Certificate name="Microsoft.WindowsAzure.Plugins.RemoteAccess.PasswordEncryption" thumbprint="C33B6C432C25581601B84C80F86EC2809DC224E8" thumbprintAlgorithm="sha1" /&gt;
&lt;/Certificates&gt;</pre>
      <p>Upload Remote Desktop certificates as a one-time setup step using the following cmdlet script:</p>
      <pre class="prettyprint">Add-Certificate -serviceName <em>&lt;HOSTEDSERVICENAME&gt;</em> -certificateToDeploy (get-item cert:\CurrentUser\MY\<em>&lt;THUMBPRINT&gt;</em>)</pre>
      <p>For example:</p>
      <pre class="prettyprint">Add-Certificate -serviceName 'mytesthostedservice' -certificateToDeploy (get-item cert:\CurrentUser\MY\C33B6C432C25581601B84C80F86EC2809DC224E8</pre>
      <p>Alternatively you can export the certificate file PFX with private key and upload certificates to each target Hosted Service using the Windows Azure Management Portal. Read the following article to learn more: <a href="http://msdn.microsoft.com/en-us/library/windowsazure/gg443832.aspx">http://msdn.microsoft.com/en-us/library/windowsazure/gg443832.aspx</a>.</p>
      <p>
        <strong>Upgrade Deployment vs. Delete Deployment -&gt; New Deployment</strong>
      </p>
      <p>The script will by default perform an Upgrade Deployment ($enableDeploymentUpgrade = 1) when no parameter is passed in or the value 1 is passed explicitly. For single instances this has the advantage of taking less time than a full deployment. For instances that require high availability this also has the advantage of leaving some instances running while others are upgraded (walking your update domain), plus your VIP will not be deleted.</p>
      <p>Upgrade Deployment can be disabled in the script ($enableDeploymentUpgrade = 0) or by passing –enableDeploymentUpgrade 0 as a parameter, which will alter the script behavior to first delete any existing deployment and then create a new deployment.</p>
      <p>
        <strong>Warning:</strong> The script will always delete or replace your existing deployments by default if they are detected. This is necessary to enable continuous delivery from automation where no user/operator prompting is possible.</p>
    </li>
  </ol>
  <h2>
    <a name="step5">
    </a>Step 5: Publish a Package using TFS Team Build (Optional)</h2>
  <p>This step will wire up TFS Team Build to the script created in step 4, which handles publishing of the package build to Windows Azure. This entails modifying the Process Template used by your build definition so that it runs a Publish activity at the end of the workflow. The Publish activity will execute your PowerShell command passing in parameters from the build. Output of the MSBuild targets and publish script will be piped into the standard build output.</p>
  <ol>
    <li>
      <p>Edit the Build Definition responsible for continuous deploy.</p>
    </li>
    <li>
      <p>Select the <strong>Process</strong> tab.</p>
    </li>
    <li>
      <p>Create a new process template by clicking on the New... button in the Process Template Selection area of the tab. The New Build Process Template dialog will be loaded. In this dialog ensure Copy an Existing XAML file is selected, and browse to the ProcessTemplate to copy if you want to change the basis from the default. Provide the new template with a name such as DefaultTemplateAzure.</p>
    </li>
    <li>
      <p>Open the selected Process Template for editing. You can open directly in the Workflow designer or in the XML editor to work with the XAML.</p>
    </li>
    <li>
      <p>Add the following list of new arguments as separate line items in the arguments tab of the workflow designer. All arguments should have direction=In and type=String. These will be used to flow parameters from the build definition into the workflow, which then get used to call the publish script.</p>
      <pre class="prettyprint">SubscriptionName
StorageAccountName
CloudConfigLocation
PackageLocation
Environment
SubscriptionDataFileLocation
PublishScriptLocation
ServiceName</pre>
      <p>
        <img src="/media/net/common-task-tfs-03.png" />
      </p>
      <p>The corresponding XAML looks like this:</p>
      <pre class="prettyprint">&lt;Activity  … /&gt;
  &lt;x:Members&gt;
    &lt;x:Property Name="BuildSettings" Type="InArgument(mtbwa:BuildSettings)" /&gt;
    &lt;x:Property Name="TestSpecs" Type="InArgument(mtbwa:TestSpecList)" /&gt;
    &lt;x:Property Name="BuildNumberFormat" Type="InArgument(x:String)" /&gt;
    &lt;x:Property Name="CleanWorkspace" Type="InArgument(mtbwa:CleanWorkspaceOption)" /&gt;
    &lt;x:Property Name="RunCodeAnalysis" Type="InArgument(mtbwa:CodeAnalysisOption)" /&gt;
    &lt;x:Property Name="SourceAndSymbolServerSettings" Type="InArgument(mtbwa:SourceAndSymbolServerSettings)" /&gt;
    &lt;x:Property Name="AgentSettings" Type="InArgument(mtbwa:AgentSettings)" /&gt;
    &lt;x:Property Name="AssociateChangesetsAndWorkItems" Type="InArgument(x:Boolean)" /&gt;
    &lt;x:Property Name="CreateWorkItem" Type="InArgument(x:Boolean)" /&gt;
    &lt;x:Property Name="DropBuild" Type="InArgument(x:Boolean)" /&gt;
    &lt;x:Property Name="MSBuildArguments" Type="InArgument(x:String)" /&gt;
    &lt;x:Property Name="MSBuildPlatform" Type="InArgument(mtbwa:ToolPlatform)" /&gt;
    &lt;x:Property Name="PerformTestImpactAnalysis" Type="InArgument(x:Boolean)" /&gt;
    &lt;x:Property Name="CreateLabel" Type="InArgument(x:Boolean)" /&gt;
    &lt;x:Property Name="DisableTests" Type="InArgument(x:Boolean)" /&gt;
    &lt;x:Property Name="GetVersion" Type="InArgument(x:String)" /&gt;
    &lt;x:Property Name="PrivateDropLocation" Type="InArgument(x:String)" /&gt;
    &lt;x:Property Name="Verbosity" Type="InArgument(mtbw:BuildVerbosity)" /&gt;
    &lt;x:Property Name="Metadata" Type="mtbw:ProcessParameterMetadataCollection" /&gt;
    &lt;x:Property Name="SupportedReasons" Type="mtbc:BuildReason" /&gt;
    &lt;x:Property Name="SubscriptionName" Type="InArgument(x:String)" /&gt;
    &lt;x:Property Name="StorageAccountName" Type="InArgument(x:String)" /&gt;
    &lt;x:Property Name="CloudConfigLocation" Type="InArgument(x:String)" /&gt;
    &lt;x:Property Name="PackageLocation" Type="InArgument(x:String)" /&gt;
    &lt;x:Property Name="Environment" Type="InArgument(x:String)" /&gt;
    &lt;x:Property Name="SubscriptionDataFileLocation" Type="InArgument(x:String)" /&gt;
    &lt;x:Property Name="PublishScriptLocation" Type="InArgument(x:String)" /&gt;
    &lt;x:Property Name="ServiceName" Type="InArgument(x:String)" /&gt;
  &lt;/x:Members&gt;

  &lt;this:Process.MSBuildArguments&gt;
</pre>
    </li>
    <li>
      <p>Add a new sequence at the end of Run On Agent:</p>
      <ol>
        <li>
          <p>Start by adding an If Statement activity to check for a valid script file. Set the condition to this value:</p>
          <pre class="prettyprint">Not String.IsNullOrEmpty(PublishScriptLocation)</pre>
        </li>
        <li>
          <p>In the Then case of the If Statement, add a new Sequence activity. Set the display name to 'Start publish'</p>
        </li>
        <li>
          <p>With the Start publish sequence still selected, add the following list of new variables as separate line items in the variables tab of the workflow designer. All variables should have Variable type =String and Scope=Start publish. These will be used to flow parameters from the build definition into the workflow, which then get used to call the publish script.</p>
          <ul>
            <li>
              <p>SubscriptionDataFilePath, of type String</p>
            </li>
            <li>
              <p>PublishScriptFilePath, of type String</p>
              <p>
                <img src="/media/net/common-task-tfs-04.png" />
              </p>
            </li>
          </ul>
        </li>
        <li>
          <p>Add a ConvertWorkspaceItem activity at the beginning of the new Sequence. Direction=ServerToLocal, DisplayName='Convert publish script filename', Input=' PublishScriptLocation', Result='PublishScriptFilePath', Workspace='Workspace'. This activity converts the path to the publish script from TFS server locations (if applicable) to a standard local disk path.</p>
        </li>
        <li>
          <p>Add another ConvertWorkspaceItem activity at the end of the new Sequence. Direction=ServerToLocal, DisplayName='Convert subscription filename', Input=' SubscriptionDataFileLocation', Result= 'SubscriptionDataFilePath', Workspace='Workspace'.</p>
        </li>
        <li>
          <p>Add an InvokeProcess activity at the end of the new Sequence. This activity calls PowerShell.exe with the arguments passed in by the Build Definition.</p>
          <ol>
            <li>
              <p>Arguments = String.Format(" -File ""{0}"" -serviceName {1} -storageAccountName {2} -packageLocation ""{3}"" -cloudConfigLocation ""{4}"" -subscriptionDataFile ""{5}"" -selectedSubscription {6} -environment ""{7}""", PublishScriptFilePath, ServiceName, StorageAccountName, PackageLocation, CloudConfigLocation, SubscriptionDataFilePath, SubscriptionName, Environment)</p>
            </li>
            <li>
              <p>DisplayName ='Execute publish script'</p>
            </li>
            <li>
              <p>FileName = "PowerShell"</p>
            </li>
            <li>
              <p style="font-size: 10px;">OutputEncoding= 'System.Text.Encoding.GetEncoding(System.Globalization.CultureInfo.InstalledUICulture.TextInfo.OEMCodePage)'</p>
            </li>
          </ol>
        </li>
        <li>
          <p>In the Handle Standard Output section textbox of the InvokeProcess, set the textbox value to 'data'. This is a variable to store the standard output data.</p>
        </li>
        <li>
          <p>Add a WriteBuildMessage activity just below the standard output section. Set the Importance = 'Microsoft.TeamFoundation.Build.Client.BuildMessageImportance.High' and the Message='data'. This ensures the standard output of the script will get written to the build output.</p>
        </li>
        <li>
          <p>In the Handle Standard Error section textbox of the InvokeProcess, set the textbox value to 'data'. This is a variable to store the standard error data.</p>
        </li>
        <li>
          <p>Add a WriteBuildError activity just below the standard output section. Set the Message='data'. This ensures the standard errors of the script will get written to the build error output.</p>
        </li>
      </ol>
      <p>The final result of the publish workflow activities will look like this in the designer:</p>
      <p>
        <img src="/media/net/common-task-tfs-05.png" />
      </p>
      <p>The final result of the publish workflow activities will look like this in XAML:</p>
      <pre class="prettyprint">	    &lt;/TryCatch&gt;

      &lt;If Condition="[Not String.IsNullOrEmpty(PublishScriptLocation)]" sap:VirtualizedContainerService.HintSize="1539,552"&gt;
        &lt;If.Then&gt;
          &lt;Sequence DisplayName="Start publish" sap:VirtualizedContainerService.HintSize="297,446"&gt;
            &lt;Sequence.Variables&gt;
              &lt;Variable x:TypeArguments="x:String" Name="PublishScriptFilePath" /&gt;
              &lt;Variable x:TypeArguments="x:String" Name="SubscriptionDataFilePath" /&gt;
            &lt;/Sequence.Variables&gt;
            &lt;sap:WorkflowViewStateService.ViewState&gt;
              &lt;scg:Dictionary x:TypeArguments="x:String, x:Object"&gt;
                &lt;x:Boolean x:Key="IsExpanded"&gt;True&lt;/x:Boolean&gt;
              &lt;/scg:Dictionary&gt;
            &lt;/sap:WorkflowViewStateService.ViewState&gt;
            &lt;mtbwa:ConvertWorkspaceItem DisplayName="Convert publish script filename" sap:VirtualizedContainerService.HintSize="234,22" Input="[PublishScriptLocation]" Result="[PublishScriptFilePath]" Workspace="[Workspace]" /&gt;
            &lt;mtbwa:ConvertWorkspaceItem DisplayName="Convert subscription data file filename" sap:VirtualizedContainerService.HintSize="234,22" Input="[SubscriptionDataFileLocation]" Result="[SubscriptionDataFilePath]" Workspace="[Workspace]" /&gt;
            &lt;mtbwa:InvokeProcess Arguments="[String.Format(" -File ""{0}"" -serviceName {1} -storageAccountName {2} -packageLocation ""{3}"" -cloudConfigLocation ""{4}"" -subscriptionDataFile ""{5}"" -selectedSubscription {6} -environment ""{7}""", PublishScriptFilePath, ServiceName, StorageAccountName, PackageLocation, CloudConfigLocation, SubscriptionDataFilePath, SubscriptionName, Environment)]" DisplayName="Execute publish script" FileName="PowerShell" sap:VirtualizedContainerService.HintSize="234,198"&gt;
              &lt;mtbwa:InvokeProcess.ErrorDataReceived&gt;
                &lt;ActivityAction x:TypeArguments="x:String"&gt;
                  &lt;ActivityAction.Argument&gt;
                    &lt;DelegateInArgument x:TypeArguments="x:String" Name="data" /&gt;
                  &lt;/ActivityAction.Argument&gt;
                  &lt;mtbwa:WriteBuildError sap:VirtualizedContainerService.HintSize="200,22" Message="[data]" /&gt;
                &lt;/ActivityAction&gt;
              &lt;/mtbwa:InvokeProcess.ErrorDataReceived&gt;
              &lt;mtbwa:InvokeProcess.OutputDataReceived&gt;
                &lt;ActivityAction x:TypeArguments="x:String"&gt;
                  &lt;ActivityAction.Argument&gt;
                    &lt;DelegateInArgument x:TypeArguments="x:String" Name="data" /&gt;
                  &lt;/ActivityAction.Argument&gt;
                  &lt;mtbwa:WriteBuildMessage sap:VirtualizedContainerService.HintSize="200,22" Importance="[Microsoft.TeamFoundation.Build.Client.BuildMessageImportance.High]" Message="[data]" mva:VisualBasic.Settings="Assembly references and imported namespaces serialized as XML namespaces" /&gt;
                &lt;/ActivityAction&gt;
              &lt;/mtbwa:InvokeProcess.OutputDataReceived&gt;
            &lt;/mtbwa:InvokeProcess&gt;
          &lt;/Sequence&gt;
        &lt;/If.Then&gt;
      &lt;/If&gt;

    &lt;/mtbwa:AgentScope&gt;
    &lt;mtbwa:InvokeForReason DisplayName="Check In Gated Changes for CheckInShelveset Builds" sap:VirtualizedContainerService.HintSize="1370,146" Reason="CheckInShelveset"&gt;
      &lt;mtbwa:CheckInGatedChanges DisplayName="Check In Gated Changes" sap:VirtualizedContainerService.HintSize="200,22" /&gt;
    &lt;/mtbwa:InvokeForReason&gt;
  &lt;/Sequence&gt;
&lt;/Activity&gt;

</pre>
    </li>
    <li>
      <p>Save the DefaultTemplateAzure workflow and Check In this file.</p>
    </li>
    <li>
      <p>Select the DefaultTemplateAzure process template in your build definition. Click Refresh or select the New button if you do not yet see this file in the list of Process Templates.</p>
    </li>
    <li>
      <p>Set the parameter property values in the Misc section as follows:</p>
      <ol>
        <li>
          <p>CloudConfigLocation = 'c:\drops\app.publish\ServiceConfiguration.Cloud.cscfg' <br /><em>This value is derived from: ($PublishDir)ServiceConfiguration.Cloud.cscfg</em></p>
        </li>
        <li>
          <p>PackageLocation = 'c:\drops\app.publish\ContactManager.Azure.cspkg' <br /><em>This value is derived from: ($PublishDir)($ProjectName).cspkg</em></p>
        </li>
        <li>
          <p>PublishScriptLocation = 'c:\scripts\WindowsAzure\PublishCloudApp.ps1'</p>
        </li>
        <li>
          <p>ServiceName = 'myhostedservicename' <br /><em>Use the appropriate hosted service name here</em></p>
        </li>
        <li>
          <p>Environment = 'Staging'</p>
        </li>
        <li>
          <p>StorageAccountName = 'mystorageaccountname' <br /><em>Use the appropriate storage account name here</em></p>
        </li>
        <li>
          <p>SubscriptionDataFileLocation = 'c:\scripts\WindowsAzure\Subscription.xml'</p>
        </li>
        <li>
          <p>SubscriptionName = 'default'</p>
        </li>
      </ol>
      <p>
        <img src="/media/net/common-task-tfs-06.png" />
      </p>
    </li>
    <li>
      <p>Save the changes to the Build Definition.</p>
    </li>
    <li>
      <p>Queue a Build to execute both the package build and publish. If you have a trigger set to Continuous Deploy, you will execute this behavior on every check-in.</p>
    </li>
  </ol>
  <h3>
    <a name="script">
    </a>PublishCloudApp.ps1 script template</h3>
  <pre class="prettyprint">Param(  $serviceName = "",
        $storageAccountName = "",
        $packageLocation = "",
        $cloudConfigLocation = "",
        
        $Environment = "Staging",
        $deploymentLabel = "ContinuousDeploy to $servicename",
        $timeStampFormat = "g",
        $alwaysDeleteExistingDeployments = 1,
        $enableDeploymentUpgrade = 1,
        $selectedsubscription = "default",
        $subscriptionDataFile = "c:\scripts\WindowsAzure\Subscription.xml"
     )
      

#initialize cmdlet snapin and subscription
if ((Get-PSSnapin | ?{$_.Name -eq "WAPPSCmdlets"}) -eq $null)
{
  Add-PSSnapin WAPPSCmdlets
}
Set-Subscription -SubscriptionDataFile $subscriptionDataFile
select-subscription $selectedsubscription

$subscription = Get-Subscription $selectedsubscription
$subscriptionname = $subscription.subscriptionname
$subscriptionid = $subscription.subscriptionid
$slot = $environment


function SuspendDeployment()
{
	write-progress -id 1 -activity "Suspending Deployment" -status "In progress"
	Write-Output "$(Get-Date –f $timeStampFormat) - Suspending Deployment: In progress"

	$suspend = Set-DeploymentStatus -Slot $slot -ServiceName $serviceName -Status Suspended
	$opstat = Get-OperationStatus -operationid $suspend.operationId
	
	while ([string]::Equals($opstat, "InProgress"))
	{
		sleep -Seconds 1

		$opstat = Get-OperationStatus -operationid $suspend.operationId
	}

	write-progress -id 1 -activity "Suspending Deployment" -status $opstat
	Write-Output "$(Get-Date –f $timeStampFormat) - Suspending Deployment: $opstat"
}

function DeleteDeployment()
{
	SuspendDeployment

	write-progress -id 2 -activity "Deleting Deployment" -Status "In progress"
	Write-Output "$(Get-Date –f $timeStampFormat) - Deleting Deployment: In progress"

	$removeDeployment = Remove-Deployment -Slot $slot -ServiceName $serviceName
	$opstat = WaitToCompleteNoProgress($removeDeployment.operationId)
	
	write-progress -id 2 -activity "Deleting Deployment" -Status $opstat
	Write-Output "$(Get-Date –f $timeStampFormat) - Deleting Deployment: $opstat"
	
	sleep -Seconds 10
}

function WaitToCompleteNoProgress($operationId)
{
	$result = Get-OperationStatus -OperationId $operationId
	
	while ([string]::Equals($result, "InProgress"))
	{
		sleep -Seconds 1
		$result = Get-OperationStatus -OperationId $operationId
	}
	
	return $result
}

function Publish()
{
	$deployment = Get-Deployment -ServiceName $serviceName -Slot $slot -ErrorVariable a -ErrorAction silentlycontinue 
    if ($a[0] -ne $null)
    {
        write-host "No deployment is detected. Creating a new deployment. "
    }
    #check for existing deployment and then either upgrade, delete + deploy, or cancel according to $alwaysDeleteExistingDeployments and $enableDeploymentUpgrade boolean variables
	if ($deployment.Name -ne $null)
	{
		switch ($alwaysDeleteExistingDeployments)
	    {
	        1 
			{
                switch ($enableDeploymentUpgrade)
                {
                    1
                    {
                        Write-Output "$(Get-Date –f $timeStampFormat) - Deployment exists in $servicename.  Upgrading deployment."
				        UpgradeDeployment
                    }
                    0  #Delete then create new deployment
                    {
                        Write-Output "$(Get-Date –f $timeStampFormat) - Deployment exists in $servicename.  Deleting deployment."
				        DeleteDeployment
                        CreateNewDeployment
                        
                    }
                } # switch ($enableDeploymentUpgrade)
			}
	        0
			{
				Write-Output "$(Get-Date –f $timeStampFormat) - ERROR: Deployment exists in $servicename.  Script execution cancelled."
				exit
			}
	    }
	} else {
            CreateNewDeployment
    }
}

function CreateNewDeployment()
{
	write-progress -id 3 -activity "Creating New Deployment" -Status "In progress"
	Write-Output "$(Get-Date –f $timeStampFormat) - Creating New Deployment: In progress"

	$newdeployment = New-Deployment -Slot $slot -Package $packageLocation -Configuration $cloudConfigLocation -label $deploymentLabel -ServiceName $serviceName -StorageAccountName $storageAccountName
	$opstat = WaitToCompleteNoProgress($newdeployment.operationId)
	
	write-progress -id 3 -activity "Creating New Deployment" -Status $opstat
    
    $completeDeployment = Get-Deployment -ServiceName $serviceName -Slot $slot
    $completeDeploymentID = $completeDeployment.deploymentid
	Write-Output "$(Get-Date –f $timeStampFormat) - Creating New Deployment: $opstat, Deployment ID: $completeDeploymentID"
    
	StartInstances
}

function UpgradeDeployment()
{
	write-progress -id 3 -activity "Upgrading Deployment" -Status "In progress"
	Write-Output "$(Get-Date –f $timeStampFormat) - Upgrading Deployment: In progress"

	Update-Deployment -Slot $slot -Package $packageLocation -Configuration $cloudConfigLocation -label $deploymentLabel -ServiceName $serviceName -StorageAccountName $storageAccountName | 
   Get-OperationStatus -WaitToComplete
    
    #$opstat = WaitToCompleteNoProgress($newdeployment.operationId)
	
	#write-progress -id 3 -activity "Upgrading New Deployment" -Status $opstat
    
    $completeDeployment = Get-Deployment -ServiceName $serviceName -Slot $slot
    $completeDeploymentID = $completeDeployment.deploymentid
	Write-Output "$(Get-Date –f $timeStampFormat) - Upgrading Deployment: Succeeded, Deployment ID: $completeDeploymentID"

}

function StartInstances()
{
	write-progress -id 4 -activity "Starting Instances" -status "In progress"
	Write-Output "$(Get-Date –f $timeStampFormat) - Starting Instances: In progress"

	$run = Set-DeploymentStatus -Slot $slot -ServiceName $serviceName -Status Running
	$deployment = Get-Deployment -ServiceName $serviceName -Slot $slot
	$oldStatusStr = @("") * $deployment.RoleInstanceList.Count
	
	while (-not(AllInstancesRunning($deployment.RoleInstanceList)))
	{
		$i = 1
		foreach ($roleInstance in $deployment.RoleInstanceList)
		{
			$instanceName = $roleInstance.InstanceName
			$instanceStatus = $roleInstance.InstanceStatus

			if ($oldStatusStr[$i - 1] -ne $roleInstance.InstanceStatus)
			{
				$oldStatusStr[$i - 1] = $roleInstance.InstanceStatus
				Write-Output "$(Get-Date –f $timeStampFormat) - Starting Instance '$instanceName': $instanceStatus"
			}

			write-progress -id (4 + $i) -activity "Starting Instance '$instanceName'" -status "$instanceStatus"
			$i = $i + 1
		}

		sleep -Seconds 1

		$deployment = Get-Deployment -ServiceName $serviceName -Slot $slot
	}

	$i = 1
	foreach ($roleInstance in $deployment.RoleInstanceList)
	{
		$instanceName = $roleInstance.InstanceName
		$instanceStatus = $roleInstance.InstanceStatus

		if ($oldStatusStr[$i - 1] -ne $roleInstance.InstanceStatus)
		{
			$oldStatusStr[$i - 1] = $roleInstance.InstanceStatus
			Write-Output "$(Get-Date –f $timeStampFormat) - Starting Instance '$instanceName': $instanceStatus"
		}

		write-progress -id (4 + $i) -activity "Starting Instance '$instanceName'" -status "$instanceStatus"
		$i = $i + 1
	}
	
	$opstat = Get-OperationStatus -operationid $run.operationId
	
	write-progress -id 4 -activity "Starting Instances" -status $opstat
	Write-Output "$(Get-Date –f $timeStampFormat) - Starting Instances: $opstat"
}

function AllInstancesRunning($roleInstanceList)
{
	foreach ($roleInstance in $roleInstanceList)
	{
		if ($roleInstance.InstanceStatus -ne "Ready")
		{
			return $false
		}
	}
	
	return $true
}


Write-Output "$(Get-Date –f $timeStampFormat) - Azure Cloud App deploy script started."
Write-Output "$(Get-Date –f $timeStampFormat) - Preparing deployment of $deploymentLabel for $subscriptionname with Subscription ID $subscriptionid."

Publish

$deployment = Get-Deployment -slot $slot -serviceName $servicename
$deploymentUrl = $deployment.Url

Write-Output "$(Get-Date –f $timeStampFormat) - Created Cloud App with URL $deploymentUrl."
Write-Output "$(Get-Date –f $timeStampFormat) - Azure Cloud App deploy script finished."
</pre>