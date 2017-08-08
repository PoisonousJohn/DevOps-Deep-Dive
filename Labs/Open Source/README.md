Оригинал: https://raw.githubusercontent.com/AzureCAT-GSI/DevOps-Deep-Dive/master/Labs/Open%20Source/README.md

# DevOps на OpenSource стеке

## Краткое содержание

В ходе лабороторной работы вы получите опыт выстраивания процессов continious integration с использованием open source инструментов. Вы начнете с конфигурации контейнеризованный среды Jenkins, а затем соберете простое Node.js приложение с помощью NPM и Gulp.

## Цели

После выполнения всех упражнений, вы сможете:

- Деплоить и настраивать Jenkins
- Использовать контейнеризацию в процессах DevOps
- Собирать опен сорсные Node.js приложения

**Примерное время выполнения лабораторной: *60* минут**

# Задание : Поднять Jenkins

## Сценарий

В ходе выполнения этого задания, вы поднимите сервер Jenkins, запущенный в Docker контейнере на виртуальной машине Ubuntu Linux в Azure.

После выполнения задания вы поймете:

-   Как создавать контейнеризированные сервера приложений на Linux.
-   Как разворачивать и настраивать Jenkins server.
-   Как использовать SSH для взаимодействия с Linux серверами.

## Разверните ресурсы с помощью ARM шаблона

1. Зайдите на портал Azure <http://portal.azure.com>.

1. Чтобы использовать Jenkins внутри Docker контейнера, нам нужно настроить виртуальную машину с предустановленным Docker Engine. Чтобы сэкономить время, мы будем использовать Azure Resource Manager (ARM) шаблон. Зайдите в репозиторий шаблонов

    <https://github.com/Azure/azure-quickstart-templates>.

1. Найдите папку **docker-simple-on-ubuntu** и зайдите в нее. Этот шаблон развернет все необходимые ресурсы в вашей подписке, чтобы завести Docker на Ubuntu.

1. Нажмите **Deploy to Azure** чтобы загрузить ARM шаблон в портал Azure

    <https://github.com/Azure/azure-quickstart-templates/tree/master/docker-simple-on-ubuntu>.

    ![image](./media/image1.png)

1. Заполните все обязательные параметры:
	- укажите имя и расположение новой Resource Group
	- имя и пароль администратора виртуалки
	- имя хранилища
	- днс имя для публичногого IP адреса
	- согласитесь с условиями использования **Legal Terms**.

1. Нажмите **Purchase** чтобы запустить разворачивание ресурсов.

    > Заметка: Обязательно запомните имя администратора и пароль, они понадобятся вам позже.

    ![image](./media/image2.png)

1. Разворачивание ресурсов будет длится приблизительно 10 минут. Чтобы не терять время, установим утилиту, которая нам понадобится: PuTTY (*если вы Linux fan, вы и так знаете что делать, переходите к следующим шагам*).
PuTTY это простой и бесплатный SSH клиент для Windows который позволит получить доступ к нашей виртуальной машине. В мире Windows, мы бы просто использовали Remote Desktop Connection, однако, наш Ubuntu Server запущен без графического интерфейса. Поэтому мы будет взаимодействовать с сервером через SSH протокол. для этого нам и понадобится PuTTY. Перейдите по ссылке <http://www.putty.org/> и нажмите **here**.

    ![image](./media/image3.png)

1. Проще всего скачать непосредственно исполняемый файл. Например, putty.exe (the SSH and Telnet client itself) 64-bit.

    ![image](./media/image4.png)

1. Этот маленький файл putty.exe &mdash; все что нужно, чтобы установить SSH соединение с виртуальной машиной. Вернитесь на портал и дождитесь, пока процесс развертывания ресурсов завершиться.

1. Перейдите в созданную Resource Group и откройте виртуальную машину.

    ![image](./media/image5.png)

1. На карточке виртуальной машины, разверните секцию **Essentials** и скопируйте **Public IP address**.

    ![image](./media/image6.png)

1. Откройте PuTTY.exe.

1. В поле **Host Name** вставьте значение **Public IP Address** и нажмите **Open**.

    ![image](./media/image7.png)

1. Нажмите **Yes** чтобы добавить ключ сервера в ваш реестр.

    ![image](./media/image8.png)

1. Используя имя и пароль админа, которые вы указали при деплое ARM шаблона, войдите на виртуальную машину. Вы должны увидеть экран приветствия с системной информацией о вашей Ubuntu машине.

1. В командной строке введите **docker** и нажмите **enter**.

    ![image](./media/image9.png)

1. Система выплюнет доку по докеру, подтверждая что Docker Engine уже установлен. Выполните команду **docker version** чтобы увидеть пример взаимодействия с Docker Engine.

    ![image](./media/image10.png)

1. Теперь, когда у нас уже есть виртуалка с Ubuntu и установленным Docker Engine, нам нужен Docker образ. Команда Jenkins публикует свой Docker образ на Docker Hub:
<https://hub.docker.com/r/jenkinsci/jenkins/>.

1. Чтобы настроить контейнер с образом Jenkins, выполните команду `docker run -d -p 80:8080 jenkinsci/jenkins`. Это команда делает несколько вещей:

    - `docker run` настраивает новый контейнер

    - `-d` значит запустить в режиме демона (сервиса). Она запускает контейнер в фоне, вместо того, чтобы перекинуть вас в консоль контейнера.

    - `-p 80:8080` привязывает виртуальный порт контейнера к порту виртуальной машины. Это позволяет иметь доступ к веб-интерфейсу Jenkins с нашей собственной машины. Если мы привяжем порт контейнера к 80 порту виртуальной машины, мы сможем постучать по публичному IP адресу без указания порта в браузере.

    - `jenkinsci/jenkins` &mdash; имя Docker образа. Так как Docker не может найти образ локально, он загружает образ с Docker Hub и передает его в созданный контейнер.

1. A long alphanumeric string displays. The ID represents the new container. To get more information about all running containers, enter **docker ps**. Note the randomly generated **Names** value for your container, as we will use it shortly. In this example, the container’s name is **grave\_hopper**.

    ![image](./media/image11.png)

1. In your browser, open a new tab and navigate to your virtual machine’s Public IP address (ex. 13.93.205.238). You should see Jenkins initialize itself and after a moment, a security check will display. Jenkins has written an unlock password in to the file system to ensure only an admin can access the server.

    ![image](./media/image12.png)

1. To unlock Jenkins, we need to read the unlock file on the file system inside of our new container. In PuTTY, execute

    **docker exec -it grave\_hopper tail /var/jenkins\_home/secrets/initialAdminPassword**.

    Remember to replace **grave\_hopper** with your container’s name.

    - `docker exec` executes a command inside of a given container.

    - `-it` means interactive mode, meaning the accompanying output will be echoed out to our PuTTY terminal.

1. The returned line, another long alphanumeric string, is the password that we need to unlock Jenkins. Copy this line.

    ![image](./media/image13.png)

1. In the browser, paste the unlock code into the **Administrator password** field and click **Continue**.

    ![image](./media/image14.png)

1. This code will unlock Jenkins, allowing us to continue configuring the server. On the **Customize Jenkins** screen, select **Install suggested plugins**. Jenkins is flexible, and relies on a wide ecosystem of plugins for a variety of functionality. Allow Jenkins a few moments to download and install the desired plugins.

    ![image](./media/image15.png)

1. Once the plugins finish installing, Jenkins will ask you to create an Admin User. Complete the form if desired, but for time it is easier to click **Continue as admin**.

    ![image](./media/image16.png)

1. On the **Getting Started** screen, click **Start using Jenkins**. Jenkins installs with a baseline configuration.

    ![image](./media/image17.png)

# Exercise : Build a Node.JS Application using Jenkins

## Scenario

In this exercise, we will introduce you to the continuous integration features of Jenkins by building a Node.js web application.extensible analytics service that monitors your live applicationextensible analytics service that monitors your live application

After completing this exercise, you will understand:

-   How to install and configure Jenkins plugins.

-   How to connect Jenkins to a source control repository.

-   How to build an application using Jenkins’ build steps.

## Install and Configure a Jenkins Plugin

1. Out of the box, Jenkins works with Java applications. In order to work with Node applications, we need to install and configure a Jenkins plugin.

1. From the homepage, click **Manage Jenkins**.

    ![image](./media/image18.png)

1. On the **Manage Jenkins** screen, click **Manage Plugins**.

    ![image](./media/image19.png)

1. You use this screen to add and remove plugins from the Jenkins server. Click the **Available** tab and in the Filter field, enter in **nodejs**. The view will trim down substantially. Select **NodeJS Plugin** and click **Install without restart** to add the plugin to your server. This plugin eases the installation of Node specific tooling and configuration.

    ![image](./media/image20.png)

1. A status screen displays. Once the status of the NodeJS Plugin changes to **Success**, the plugin is available.

    ![image](./media/image21.png)

1. We need to configure the plugin. Click **Manage Jenkins**.

    ![image](./media/image22.png)

1. On the **Manage Jenkins** screen, click **Global Tool Configuration**.

    ![image](./media/image23.png)

1. In the **Global Tool Configuration** screen, we can select which version of the Node.js runtime we would like to install on the server. Scroll to the **NodeJS** section and click **Add** **NodeJS**. In the **Name** field, enter **Node6** and select **Install automatically**.

    ![image](./media/image24.png)

1. Now that the installation has a name, we need to define the actual software version. Click **Add Installer** and select **Install from nodejs.org**.

    ![image](./media/image25.png)

1. Select the latest version available and click **Save**.

    ![image](./media/image26.png)

1. Jenkins now has the latest version of Node.js installed, which you can use for continuous integration.

## Create a New Job

1. In the Jenkins world, a project is roughly analogous to a build definition in Visual Studio Team Services. It is within the item that we configure build settings and create build steps. From the Jenkins homepage, click **New Item**.

    ![image](./media/image27.png)

1. In the **Enter an item name** field, enter **NodeApp** and click **Freestyle project**.

    ![image](./media/image28.png)

1. In order to build code, we need to *have* some code. Jenkins supports Git and Subversion, two very standard repository technologies.
    We will use a repository on GitHub with a sample Node.js application as our repository. In the **Source Code Management** section, select Git, paste **https://github.com/stevenfollis/express** in the **Repository URL** field, and click **Save**.

    ![image](./media/image29.png)

1. In the **Project NodeApp** screen, we will kick off a build to see if the code retrieves successfully. Click Build Now. A build starts and can be monitored in the **Build History** widget. When it is complete, click the build number.

    ![image](./media/image30.png)

1. In the **Build Details** screen, how long the build took, when it ran, and a variety of other information displays. Click **Console Output** to see verbose control output for the particular build.

    ![image](./media/image31.png)

1. Examine how the server works with the Git repository, cloning from GitHub into the project’s workspace.

    ![image](./media/image32.png)

We have successfully connected a source repository and executed our first build. To provide more value, we need to actually create build steps that interact with our code.

## Creating Custom Build Steps

1. We will create some build steps to supercharge our workflow. Hover over **NodeApp** and selecting **Configure**.

    ![image](./media/image33.png)

1. Scroll down the **Configuration** screen to the **Build Environment** section.

1. Select **Provide Node & npm bin/ folder to PATH** allowing us to use those commands in our build scripts.

1. In the **Build** section, click **Add build step** and select **Execute shell**. We can select a variety of scripts, but shells are often the most standard way of executing commands.

    ![image](./media/image34.png)

1. In the **Command** field, we will configure our NPM actions. First, we want to check the version of NPM installed and then retrieve all dependencies using the following code:

    ```
    npm --version

    npm install
    ```

    ![image](./media/image35.png)

1. We will then create a second build step to execute our Gulp related activities. First, we will install Gulp globally, check its version, and execute the Gulpfile’s default task. Click **Add build step**, select **Execute shell**, and enter:

    ```
    npm install -g gulp

    gulp --version

    gulp default
    ```

    ![image](./media/image36.png)

1. Once the two tasks are configured, click **Save** and **Build Now**. When the build finishes, click its number to see the detailed screen.

    ![image](./media/image37.png)

1. Examine the **Console Output** for this build, noticing all of the activity happening across NPM and Gulp.

    ![image](./media/image38.png)

1. We will see what the build did. Hover over the NodeApp project title, click the **Workspace** link.

    ![image](./media/image39.png)

1. In the **Workspace**, we can see of the files cloned in from the original GitHub repository and a series of additional files that were created in our build steps. The **node\_modules** folder represents all of our project dependencies, whereas the **deploy** folder was generated from Gulp. In our Gulp tasks, we are zipping up the entire contents of the folder into a webdeploy package that we can deploy to an environment such as an Azure Web App. Click the **deploy** folder.

    ![image](./media/image40.png)

1. Click **deployment-package.zip** to download it to your local machine, examining the file contents. You can deploy this package to a variety of different environments and it represents our fully built application.

The activity is complete. You took a running Jenkins environment, created a new build project, connected it to a source code repository, defined custom build steps, and executed a series of build activities, all hosted inside of Azure. From here, you could take the webdeploy package and push it to environments as part of a further continuous deployment pipeline.

# Review

You have now completed the Open Source on Microsoft Azure Lab. In this lab you created a Linux virtual machine running in Microsoft Azure, accessed it via SSH on the command line, operated Docker to install the Jenkins image from Docker Hub, configured Jenkins, and built a Node.js application. The webdeploy package could then be taken and deployed to an environment of your choosing.