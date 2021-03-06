# Creating Databases in Compute Engine

## Overview

In this lab, you will create database servers running on Google Cloud Compute Engine. You will create both Linux and Windows servers and will use the CLI to automate the creation of a server. 

### Objectives

In this lab, you will learn how to perform the following tasks:

*   Create a MySQL database on Linux
*   Create a SQL Server database on Windows
*   Automate server creation using the Google Cloud SDK

## Task 0. Lab Setup

In this task, you use Qwiklabs and perform initialization steps for your lab.

### Access Qwiklabs

![[/fragments/startqwiklab]]

After you complete the initial sign-in steps, the project dashboard appears.

![GCP Project Dashboard](img/gcpprojectdashboard.png)

Click __Select a project__, highlight your _GCP Project ID_, and click
__OPEN__ to select your project.

![[/fragments/cloudshell]]

## Task 1. Create a MySQL database on Linux

1.  In the Navigation menu ( ![Menu](img/menu.png) ), click on **Compute Engine > VM Instances**.

2.  Click on the **Create** button. 

3.  Name the instance `mysql-db`. <p>In the **Boot disk** section, click the **Change** button. 

![Change](img/boot-disk-change.png)


4.  Select **Ubuntu** from the Operating system dropdown, and from the Version dropdown, select **Ubuntu 16.04 LTS**. Click the **Select** button. <p>Accept all the remaining defaults and click the **Create** button at the bottom of the form.</p>

5.  It will take a few seconds for the VM to be ready. When it is, click on the **SSH** button. This will log you into the server in a new tab.

![Change](img/ssh-button.png)

6.  In the terminal window, enter the following to update the packages and install MySQL using the Apt package manager. During the installation, you will be prompted to enter a password for the root database user. Enter something you will remember as you will need it later.

```
sudo apt update
sudo apt install -y mysql-server
```

7.  Enter the following command to secure the database.

```
sudo mysql_secure_installation
```

8.  Enter the password you created a moment ago. Read each prompt, but say `No` to all of them.

9.  Now, let's try logging into the database. Enter the following command and when prompted enter your password.

```
sudo mysql -u root -p
```

10.  You should now have a `mysql>` prompt. Enter the following command to see the current databases.

```
SHOW databases; 
```

11.  Create a new database with the following command.

```
CREATE database petsdb; 
```

12.  Now switch to that database.

```
USE petsdb; 
```

13.  Now create a table.

```
CREATE TABLE pets (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255),
    breed VARCHAR(255)
);
```

14. Add a record.

```
INSERT INTO pets (name, breed) 
VALUES ('Noir', 'Schnoodle');
```

15. See if your record was added.

```
SELECT * FROM pets;
```

16. Type `exit` to log out of the database. Type `exit` again to close the SSH session.

## Review

You just create a MySQL database server using Ubuntu Linux. 

## Task 2. Create a SQL Server database on Windows

1.  Return the Google Cloud console and the Compute Engine service.

2.  Click on the **Create Instance** button at the top.

![Create Instance](img/create-instance-button.png)


3.  Name this instance `sql-server-db`. In the **Machine type** dropdown, select `n1-standard-4`. <p>In the Boot disk section, click the **Change** button. Select `SQL Server on Windows` from the **Operating system** dropdown and from the **Version** dropdown, select `SQL Server 2019 Web...` edition. Click the **Select** button</p><p>Accept the remaining defaults and then click the **Create** button at the bottom.

4.  It will take a couple minutes for the Windows server to be ready. When it is click the dropdown arrow next to the RDP button and select **Set Windows password**.

![Set Windows PW](img/set-win-pw.png)

<aside><p><strong>Note: </strong>If you get to this point too fast, the Windows server may not be ready even though there is a green check in the console indcating it is. Wait a couple minutes and try again.</p></aside>

5.  Change the Username to your first name and click the **Set** button. Copy and paste the generated password into a text file so you don't lose it. 

6.  You need to have an RDP client to log onto the Windows machine. <p>If you are using Windows, you already have RDP. Just click the down arrow next to the **RDP** button and select **Download the RDP file**. Once it's downloaded, double-click on it and try logging on with the your username and the password you copied. </p><p>If you are using a Mac you can install the **Microsoft Remote Desktop** client from the Apple App Store (*you may already have it installed*). Once you install the client, download the RDP file and try logging in with your username and the generated password.</p><p>Lastly, you can install the Chrome RDP extension. Click the RDP button and follow the instructions. Once you have an RDP client setup try logging in with your username and password.

<aside><p><strong>Note:</strong>If you get to this point too fast the Windows server may not be ready even though there is a green check in the console indcating it is. If the server connecton times out, give it a couple minutes and try again.</p></aside>

7.  Once you get logged onto the Windows server, you can close the Server Manager Dashboard.

8.  Click on the Start menu and expand the **Microsoft SQL Server 2019** folder, then select **SQL Server 2019 Installation Center**. You may be prompted with a popup asking if you want to allow this app to make changes to your device. Answer yes.

9.  Click on the **Installation** page in the resulting program and then select Install SQL Server Management tools. 

10.  This opens a web page in Internet Explorer. You will be given mutiple security warnings in Internet Explorer. Add each site to the list of trusted sites. If you are prompted to use recommended security settings for Internet Explorer, just click the **Ask me later** button. Click the **Download SQL Server Management Studio** link. Again you might have to add this site to the list of trusted sites. After adding the site, you may have to click the download link again, it may also simply say **Download SSMS**. If prompted tell it to save it.

11.  Once the installation program is downloaded, run it and follow the instructions. <p>When the installation completes, click on the Start Menu and type `SSMS`. Click on the **SQL Server Management Studio** shortcut to run the program.</p> Also, close the browser and the **SQL Server 2019 Installation Center**.

12.  In the Connect to Server dialog, the Server name should be `SQL-SERVER-DB`. Click the **Connect** button.

13.  In Management Studio, find the Object Explorer on the left. Right-click on the `SQL-SERVER-DB` at the top and select **Properties**. <p>In the Server Properties dialog, select the **Security** page.</p>

14.  In the **Server Authentication** section, select the **SQL Server and Windows Authentication mode** radio button. Then, click the **OK** button. Read the warning message and click **OK** again.

15.  Right-click on the server again in Object Explorer and select **Restart**. Choose **Yes** when prompted. 

16.  Once the server restarts, expand the **Security** and **Logins** branch of the tree view. Right-click on the Logins branch and select **New Login**. 

17.  Enter your first name in the **Login name** text box. Click on the **SQL Server authentication** radio button. Enter a password you will remember (*paste it in a text file if you like*).<p>Uncheck the **Enforce password policy** button.<p>Uncheck the **User must change password at next login** button.<p> Click on the **Server roles** page. Select the `sysadmin` role.</p><p>Click on the **User Mapping** page. Select all the databases.</p><p>Click the OK button to create the Login.</p>

18.  Let's test the login you just created. In Object Explorer, click the **Connect** button and select **Database Engine**. Select **SQL Server Authentication** from the Authentication dropdown. Enter your username and password and click **Connect**. It should work.

19.  Click the **New Query** button in the toolbar. Enter the following SQL to create a database and click the **Execute** button. 

```
CREATE DATABASE petsdb; 
```

20.  In the databases dropdown to the left of the Execute button, select your `petsdb` database. Now enter the following to create a table. Highlight just the following commands, making sure to exclude the CREATE DATABASE petsdb, and execute it. 

```
USE petsdb;
CREATE TABLE pets (
    id INT PRIMARY KEY IDENTITY (1, 1),
    name VARCHAR (MAX),
    breed VARCHAR (MAX)
); 
```

21.  Add a record.

```
INSERT INTO pets (name, breed)
VALUES ('Noir', 'Schnoodle');
```

22.  See if it worked.

```
SELECT * FROM pets; 
```

23.  Close your RDP connect.

## Review

You just create a SQL Server database server running on Windows. 

## Task 3. Automate server creation using the Google Cloud SDK

1.  Go back to the Google Cloud web console and the Compute Engine service.

2.  Click on the **Create Instance** button. 

3.  Name the instance `db-server`. <p>In the boot disk section, click the **Change** button. In the **Version** dropdown select `Debian GNU/Linux 9` and click the **Select** button.</p><p>Expand the **Management, security, disks, networking, sole tenancy section**. Paste the following in the Startup script text box. 

```
apt-get update
apt-get install -y mysql-server
```

4.  Scroll to the bottom, but ***do not*** click on the Create button, instead click on the **command line** link at the bottom.

![Command Line](img/cmd-line.png)

5.  The resulting screen shows you the gcloud CLI command you can use to automate this server's creation as you configured it.</p><p>Copy the command to the clipboard and close the popup.</p> 

6.  Click on the **Activate Cloud Shell** icon in the upper right of the console. ![Cloud Shell](img/CloudShell.png) <p>The Cloud Shell terminal will open in a pane at the bottom.</p>

7.  Paste the command to create the machine and hit **Enter**.

8.  Wait a couple minutes for the startup script to run and then in the Cloud Shell terminal enter the following to log onto the machine.

```
gcloud compute ssh db-server --zone=us-central1-a
```

9.  To see if you're database server is running enter the following command. 

```
sudo systemctl status mysql
```

<aside><p><strong>Congratulations!</strong> You created database servers running on Google Cloud Compute Engine. You created both Linux and Windows servers and have used the CLI to automate the creation of a server. </p></aside>

![[/fragments/endqwiklab]]

![[/fragments/copyright]]
