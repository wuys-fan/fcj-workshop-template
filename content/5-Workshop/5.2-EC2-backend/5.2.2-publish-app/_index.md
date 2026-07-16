---
title: "Package Application"
weight: 2
chapter: false
pre: " <b> 5.2.2 </b> "
---

# Package Spring Boot Application for Deployment

Before deploying the source code to an Amazon EC2 instance, we need to compile (build) and package the entire Java Spring Boot application into a single executable file (Executable JAR).

#### Step 1: Open Terminal in VS Code

Open your backend project using **Visual Studio Code**.
Press `` Ctrl + ` `` (or navigate to **Terminal** → **New Terminal** in the top menu) to open the integrated command-line interface within VS Code.

Ensure the terminal is pointing to the root directory of your project (where the `pom.xml` file is located).

#### Step 2: Build the project using Maven

In the VS Code Terminal, enter the following command to clean old builds, compile new source code, and package the application. We use the `-DskipTests` flag to bypass local testing, significantly speeding up the build process:

```bash
mvn clean package -DskipTests
```

After pressing Enter, Maven will download necessary dependencies and package the app. Once you see the green **[INFO] BUILD SUCCESS** message, the process is successfully completed!

![Terminal Build Success in VS Code](/images/5-Workshop/build-backend-2.png)

#### Deployment Package Ready! 📦

The execution of the command automatically generates a folder named `target`, visible in the left Explorer pane of VS Code. In the Spring Boot ecosystem, your primary focus is this single file:

- ✅ `petshop-backend-1.0.0.jar` - The Fat JAR file ready to be uploaded to your EC2 server.
- ✅ Includes an Embedded Web Server (Tomcat).
- ✅ All necessary dependencies are bundled inside.
- ✅ Configurations from `application.properties` are accurately mapped.

**File Location:**

![Maven Build target Folder](/images/1-Worklog/maven_build_output.png)

#### Common Issues

**Issue:** Error stating `mvn: command not found`
**Solution:** 
- Maven is either not installed on your machine or not configured in your Environment Variables. You need to install it and add the Maven `bin` directory path to your `PATH` variable.

**Issue:** Application throws a Java version mismatch error (e.g., requires Java 17)
**Solution:**
- Verify the specified Java version in your `pom.xml`: `<java.version>17</java.version>`.
- Ensure that the AWS EC2 instance you are about to provision also runs Java 17.

**Issue:** Database connection error during the build process
**Solution:**
- If you omitted the `-DskipTests` flag, Spring Boot attempts to connect to a local database for Unit Testing. Ensure you include this flag in your build command.

#### Next Steps

Now that we have the complete executable `.jar` file, the next phase is configuring the network and provisioning **[Amazon EC2 to launch the Server](../5.2.3-deploy-ec2/)**!