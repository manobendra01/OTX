# OTX
1. Determine Integration Path
Since the framework doesn’t have a specific extensions or lib folder, integration should rely on:

Adding custom .NET assemblies or libraries (e.g., .dll) to the main directory.
Configuring the OpenTestFramework.exe.config to reference the new library.
Using existing .NET libraries or tools like IKVM.NET to bridge Java and .NET.
2. Set Up a Java Extension Integration
If your goal is to run Java code in this .NET environment, follow these steps:

Step 2.1: Convert Java Code to .NET Assembly
Install IKVM.NET:

Download from IKVM.NET.
Extract the tools to a directory (e.g., C:\ikvm).
Write Your Java Code: Create a Java class that implements the required logic. Example:

java
Copy code
public class ExternalServiceExtension {
    public String callExternalService(String inputValue) {
        return "Response from service: " + inputValue;
    }
}
Compile the Java Code: Compile the Java file into a .class file:

bash
Copy code
javac ExternalServiceExtension.java
Package as a JAR File: Package the .class file into a JAR file:

bash
Copy code
jar cf ExternalServiceExtension.jar ExternalServiceExtension.class
Convert JAR to .NET DLL: Use IKVM to convert the JAR file into a .NET assembly:

bash
Copy code
ikvmc -target:library -out:ExternalServiceExtension.dll ExternalServiceExtension.jar
Step 2.2: Deploy the .NET Assembly
Copy the DLL to the OTF Directory: Place the generated ExternalServiceExtension.dll in the main directory:

bash
Copy code
C:\Program Files (x86)\OpenTestSystem\Open Test Framework\
Update OpenTestFramework.exe.config: Add a reference to the new DLL in the configuration:

xml
Copy code
<configuration>
    <runtime>
        <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
            <dependentAssembly>
                <assemblyIdentity name="ExternalServiceExtension" publicKeyToken="null" culture="neutral" />
                <codeBase version="1.0.0.0" href="ExternalServiceExtension.dll" />
            </dependentAssembly>
        </assemblyBinding>
    </runtime>
</configuration>
Restart OTF: Restart the application to load the new DLL.

Step 2.3: Modify the OTX File
Modify your OTX file to call the new extension:

xml
Copy code
<action name="CallExternalService" id="Action_CallExternalService">
    <realisation xsi:type="unknownExtension:ExternalServiceExtension">
        <unknownExtension:input xsi:type="StringVariable" valueOf="Variable1" />
        <unknownExtension:output xsi:type="StringVariable" name="ServiceResponse" />
    </realisation>
</action>
3. Alternative: Expose Java Logic as a Web Service
If converting Java to a .NET DLL is too complex or not feasible, you can expose the Java logic as a REST or SOAP service.

Step 3.1: Create a RESTful Web Service
Write a Simple Java REST Service: Example using SparkJava:

java
Copy code
import static spark.Spark.*;

public class ExternalService {
    public static void main(String[] args) {
        post("/api/service", (req, res) -> {
            String input = req.body();
            return "Response from service: " + input;
        });
    }
}
Run the Service: Compile and run the service:

bash
Copy code
javac ExternalService.java
java ExternalService
Step 3.2: Modify OTX to Call the Service
Use the OTX HTTP action to call the REST API:

xml
Copy code
<action name="HttpCall" id="Action_HttpCall">
    <realisation xsi:type="HttpAction">
        <HttpAction:url xsi:type="StringLiteral" value="http://localhost:4567/api/service" />
        <HttpAction:method xsi:type="HttpAction:MethodLiteral" value="POST" />
        <HttpAction:body xsi:type="StringVariable" valueOf="Variable1" />
        <HttpAction:response xsi:type="StringVariable" name="ServiceResponse" />
    </realisation>
</action>
Ensure the service is running and accessible.

4. Testing and Verification
Step 4.1: Test the DLL Integration
Restart the OTF application.
Load the updated OTX file.
Run the procedure and check the output.
Step 4.2: Test the Web Service
Ensure the Java REST service is running.
Run the OTX procedure that calls the REST API.
Verify the response is returned correctly.
5. Troubleshooting
DLL Not Loaded:

Check the .config file for correct paths and names.
Ensure the DLL is placed in the correct directory.
Service Not Accessible:

Verify the Java REST service is running and accessible from the machine.
Debugging:

Add logs to both Java and OTX to trace issues.
