# Create a simple PowerShell runbook in Azure Automation

Create an Azure Automation Account and a simple "HelloWorld" runbook that you will test and publish while you learn how to track the status of the runbook job.
Then modify the runbook to manage Azure resources (adding an authentication to Azure), in this case stoping an Azure virtual machine.

## Create an Azure Automation Account

1. Click the **Create a resource** button found on the upper left-hand corner in Azure portal

2. Type *automation*, and then select **Automation** and click the **Create** button.

3. Enter the account information. 
  For **Create Azure Run As account**, choose **Yes** so that the artifacts to simplify authentication to Azure are enabled automatically. 
  When complete, click **Create**, to start the Automation account deployment.
4. When the deployment has completed, click **All Services**, select **Automation Accounts** and select the Automation Account you created.

## Create a new runbook

Start by creating a simple runbook that outputs the text *Hello, World!*.

1. Click **Runbooks** under **Process Automation** to open the list of runbooks.
2. Create a new runbook by clicking the **+ Create a runbook** button
3. Give the runbook the name *LabRunbook*.
4. Select **Powershell** for **Runbook type**.
5. Click **Create** to create the runbook and open the textual editor.

## Add code to the runbook

1. Runbook is currently empty, type *Write-Output "Hello, World!"* in the body of the script.
2. Save the runbook by clicking **Save**.

Before you publish the runbook to make it available in production, you want to test it to make sure that it works properly.
When you test a runbook, you run its **Draft** version and view its output interactively.

1. Click **Test pane** to open the Test pane.
2. Click **Start** to start the test.
3. A runbook job is created and its status displayed.

   The job status starts as *Queued* indicating that it's waiting for a runbook worker in the cloud to come available.
   It moves to *Starting* when a worker claims the job, and then *Running* when the runbook actually starts running.

4. When the runbook job completes, its output is displayed.
5. You should see *Hello, World!*.
6. Close the Test pane to return to the canvas.

## Publish and start the runbook

The runbook that you created is still in Draft mode.
It must be published before you can run it in production.
When you publish a runbook, you overwrite the existing Published version with the Draft version.
In your case, you don't have a Published version yet because you just created the runbook.

1. Click **Publish** to publish the runbook and then **Yes** when prompted.
2. Runbook blade shows that **Status** of the runbook is **Published**.
3. Scroll back to the right to view the pane for **LabRunbook**.
4. You want to start the runbook, so click **Start** and then click **Ok** when the Start Runbook page opens.
5. A job page is opened for the runbook job that you created.
You can close this pane, but in this case you leave it open so you can watch the job's progress.
6. The job status is shown in **Job Summary** and matches the statuses that you saw when you tested the runbook.
7. Once the runbook status shows *Completed*, under **Overview** click **Output**. The Output pane is opened, and you can see *Hello, World!*.
8. Close the Output page.
9. Click **All Logs** to open the Streams pane for the runbook job.
   You should only see *Hello, World!* in the output stream, but this output can show other streams for a runbook job such as Verbose and Error if the runbook writes to them.

## Add authentication to manage Azure resources

To manage Azure resources you need to authenticate using a Run As connection that is automatically created when you create your automation account.

1. Click **Runbooks** under **Process Automation** to open the list of runbooks.
2. Click **AzureAutomationTutorialScript** (PowerShell type) and then **View** button
3. Copy the authentication code snippet (lines 10 to 32)

 ```powershell
$connectionName = "AzureRunAsConnection"
try
{
    # Get the connection "AzureRunAsConnection "
    $servicePrincipalConnection=Get-AutomationConnection -Name $connectionName

"Logging in to Azure..."
    Add-AzureRmAccount `
        -ServicePrincipal `
        -TenantId $servicePrincipalConnection.TenantId `
        -ApplicationId $servicePrincipalConnection.ApplicationId `
        -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint 
}
catch {
    if (!$servicePrincipalConnection)
    {
        $ErrorMessage = "Connection $connectionName not found."
        throw $ErrorMessage
    } else{
        Write-Error -Message $_.Exception
        throw $_.Exception
    }
}
 ```

4. Open the textual editor by clicking **Edit** on the LabRunbook page.
5. Delete the **Write-Output "Hello, World!"** line
6. Paste the code you've copied earlier and save the runbook.
7. Click **Test pane** so that you can test the runbook.
8. Click **Start** to start the test.
Once it completes, you should receive output displaying basic information from your account.
This output confirms that the Run As Account is valid.

## Add code to stop a virtual machine

Now that your runbook is authenticating to your Azure subscription, you can manage resources.
You'll add a command to stop a virtual machine you've created in Lab 04.

1. At the end of LabRunbook, add `Stop-AzureRmVM -Name "NameOfYourVM" -ResourceGroupName 'NameOfYourResourceGroup'`.
2. Save the runbook and then click **Test pane** so that you can test it.
3. Click **Start** to start the test. Once it completes, check that the virtual machine was stopped.

> [!WARNING]
> In Azure Automation, testing is **live**.
> When you test a runbook, it runs targeting the actual Azure resources.
