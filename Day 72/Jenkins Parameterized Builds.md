# Day 72: Jenkins Parameterized Builds

## Task Requirements

A new DevOps Engineer has joined the team and he will be assigned some Jenkins related tasks. Before that, the team wanted to test a simple parameterized job to understand basic functionality of parameterized builds. He is given a simple parameterized job to build in Jenkins.

### Objectives

1. Access Jenkins UI and login (username: `admin`, password: `Adm!n321`)
2. Create a parameterized job named `parameterized-job`
3. Add a string parameter named `Stage` with default value `Build`
4. Add a choice parameter named `env` with choices: Development, Staging, and Production
5. Configure job to execute a shell command that echoes both parameter values
6. Build the Jenkins job at least once with choice parameter value `Production`

---

## Solution

### Step 1: Access Jenkins UI

1. Click on the Jenkins button in the top bar
2. Log in with credentials:
   - **Username**: `admin`
   - **Password**: `Adm!n321`

### Step 2: Create a New Parameterized Job

1. From the Jenkins dashboard, click **New Item**
2. Enter the job name: `parameterized-job`
3. Select **Freestyle project**
4. Click **OK**

### Step 3: Configure String Parameter

In the job configuration page:

1. Check the box **This project is parameterized**
2. Click **Add Parameter** → **String Parameter**
3. Configure the string parameter:
   - **Name**: `Stage`
   - **Default Value**: `Build`
   - **Description**: `Stage of the build process` (optional)

### Step 4: Configure Choice Parameter

1. Click **Add Parameter** → **Choice Parameter**
2. Configure the choice parameter:
   - **Name**: `env`
   - **Choices**: (enter each on a new line)
     ```
     Development
     Staging
     Production
     ```
   - **Description**: `Target environment for deployment` (optional)

### Step 5: Configure Build Steps

1. Scroll to the **Build** section
2. Click **Add build step** → **Execute shell**
3. Enter the following shell script:
   ```bash
   echo "Stage: $Stage"
   echo "Environment: $env"
   ```

Alternatively, you can add more detailed output:

```bash
#!/bin/bash
echo "=================================="
echo "Jenkins Parameterized Build"
echo "=================================="
echo "Stage Parameter: $Stage"
echo "Environment Parameter: $env"
echo "Build Number: $BUILD_NUMBER"
echo "Build Date: $(date)"
echo "=================================="
```

### Step 6: Save the Configuration

1. Click **Save** at the bottom of the page
2. You will be redirected to the job's main page

### Step 7: Build the Job with Parameters

1. On the job page, click **Build with Parameters** (instead of the regular "Build Now")
2. You will see the parameter input form:
   - **Stage**: Leave as default `Build` or enter a different value
   - **env**: Select `Production` from the dropdown
3. Click **Build**

### Step 8: Verify Build Success

1. Check the **Build History** panel on the left
2. Click on the latest build number (e.g., `#1`)
3. Click **Console Output**
4. Verify the output shows:
   ```
   Stage: Build
   Environment: Production
   ```

---

## Example Console Output

```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/parameterized-job
[parameterized-job] $ /bin/sh -xe /tmp/jenkins1234567890123456.sh
+ echo Stage: Build
Stage: Build
+ echo Environment: Production
Environment: Production
Finished: SUCCESS
```

With the enhanced script:

```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/parameterized-job
[parameterized-job] $ /bin/sh -xe /tmp/jenkins1234567890123456.sh
+ echo ==================================
==================================
+ echo Jenkins Parameterized Build
Jenkins Parameterized Build
+ echo ==================================
==================================
+ echo Stage Parameter: Build
Stage Parameter: Build
+ echo Environment Parameter: Production
Environment Parameter: Production
+ echo Build Number: 1
Build Number: 1
++ date
+ echo Build Date: Tue Nov 26 10:30:45 UTC 2025
Build Date: Tue Nov 26 10:30:45 UTC 2025
+ echo ==================================
==================================
Finished: SUCCESS
```

---

## Understanding Parameterized Builds

### What are Parameterized Builds?

Parameterized builds allow you to pass dynamic values to your Jenkins jobs at runtime, making them flexible and reusable. Instead of hardcoding values, you can:

- Use different configurations for the same job
- Deploy to different environments
- Build different branches or versions
- Customize build behavior based on input

### Types of Parameters

| Parameter Type         | Description                 | Use Case                                       |
| ---------------------- | --------------------------- | ---------------------------------------------- |
| **String Parameter**   | Free-text input             | Version numbers, branch names, custom messages |
| **Choice Parameter**   | Dropdown selection          | Environment selection, build types             |
| **Boolean Parameter**  | Checkbox (true/false)       | Enable/disable features, skip tests            |
| **File Parameter**     | File upload                 | Configuration files, certificates              |
| **Password Parameter** | Masked text input           | API keys, credentials (use with caution)       |
| **Multi-line String**  | Text area input             | Long descriptions, scripts                     |
| **Run Parameter**      | Select from previous builds | Promote specific builds                        |

### Best Practices

1. **Use Descriptive Names**: Make parameter names clear and self-explanatory
2. **Set Sensible Defaults**: Provide default values for common scenarios
3. **Add Descriptions**: Help users understand what each parameter does
4. **Validate Input**: Add validation in your scripts to check parameter values
5. **Document Parameters**: Include parameter documentation in job description

---

## Advanced Examples

### Example 1: Multiple Parameters with Validation

```bash
#!/bin/bash

# Validate Stage parameter
if [[ ! "$Stage" =~ ^(Build|Test|Deploy)$ ]]; then
    echo "ERROR: Invalid Stage value. Must be Build, Test, or Deploy"
    exit 1
fi

# Validate env parameter
if [[ ! "$env" =~ ^(Development|Staging|Production)$ ]]; then
    echo "ERROR: Invalid environment value"
    exit 1
fi

echo "✓ Parameters validated successfully"
echo "Stage: $Stage"
echo "Environment: $env"

# Perform actions based on parameters
case $Stage in
    Build)
        echo "Building application for $env..."
        ;;
    Test)
        echo "Running tests for $env environment..."
        ;;
    Deploy)
        echo "Deploying to $env environment..."
        ;;
esac
```

### Example 2: Using Parameters in Conditional Logic

```bash
#!/bin/bash

echo "Stage: $Stage"
echo "Environment: $env"

# Different actions based on environment
if [ "$env" = "Production" ]; then
    echo "⚠️  Production deployment - enabling extra checks"
    echo "Running security scan..."
    echo "Running performance tests..."
    echo "Notifying stakeholders..."
elif [ "$env" = "Staging" ]; then
    echo "Staging deployment - running standard tests"
    echo "Running integration tests..."
elif [ "$env" = "Development" ]; then
    echo "Development deployment - fast deployment mode"
    echo "Skipping optional tests..."
fi

echo "Deployment to $env completed successfully!"
```

### Example 3: Using Boolean Parameters

Add a Boolean Parameter:

- **Name**: `RunTests`
- **Default**: Checked
- **Description**: `Run automated tests after build`

Script:

```bash
#!/bin/bash

echo "Stage: $Stage"
echo "Environment: $env"

if [ "$RunTests" = "true" ]; then
    echo "Running automated tests..."
    echo "✓ Unit tests passed"
    echo "✓ Integration tests passed"
else
    echo "Skipping tests as per user request"
fi
```

---

## Testing Different Scenarios

### Test Case 1: Default Values

- **Stage**: `Build` (default)
- **env**: `Development`
- **Expected**: Should echo both values successfully

### Test Case 2: Production Environment

- **Stage**: `Build`
- **env**: `Production`
- **Expected**: Should echo both values successfully

### Test Case 3: Custom Stage Value

- **Stage**: `Deploy`
- **env**: `Staging`
- **Expected**: Should echo custom stage value

### Test Case 4: Empty Stage (if allowed)

- **Stage**: (leave empty)
- **env**: `Development`
- **Expected**: Should use empty value or fail gracefully

---

## Troubleshooting

### Issue 1: Parameter Values Not Showing in Console Output

**Symptoms**: Echo commands don't display parameter values

**Solution**:

- Ensure parameter names match exactly (case-sensitive)
- Check that "This project is parameterized" is enabled
- Use correct variable syntax: `$Stage` or `${Stage}`
- Verify parameters are defined before the build step

### Issue 2: Build with Parameters Button Missing

**Symptoms**: Only "Build Now" button is visible

**Solution**:

- Ensure "This project is parameterized" checkbox is checked
- Save the job configuration
- Refresh the browser page
- Check if you have proper permissions

### Issue 3: Choice Parameter Not Showing All Options

**Symptoms**: Dropdown only shows one option

**Solution**:

- Ensure each choice is on a new line (press Enter after each)
- No extra spaces before or after choice values
- Save and refresh the page

### Issue 4: Parameters Not Available in Shell Script

**Symptoms**: Variables are empty or undefined

**Solution**:

- Parameters are automatically available as environment variables
- Check the parameter name spelling and case
- Use `env` command to list all available variables:
  ```bash
  echo "All available variables:"
  env | grep -E '(Stage|env)'
  ```

---

## Integration with Other Jenkins Features

### Using Parameters with Git

```bash
#!/bin/bash

# Clone specific branch based on parameter
BRANCH=${Stage:-master}
echo "Checking out branch: $BRANCH"
git checkout $BRANCH
```

### Using Parameters with Docker

```bash
#!/bin/bash

# Build Docker image with environment-specific tag
IMAGE_NAME="myapp:${env}-${BUILD_NUMBER}"
echo "Building Docker image: $IMAGE_NAME"
docker build -t $IMAGE_NAME .

if [ "$env" = "Production" ]; then
    echo "Pushing to production registry..."
    docker push registry.prod.example.com/$IMAGE_NAME
fi
```

### Using Parameters with Notifications

```bash
#!/bin/bash

# Send notifications based on environment
if [ "$env" = "Production" ]; then
    echo "Sending production deployment notification..."
    # curl -X POST slack-webhook-url -d "Deployed to Production: Stage=$Stage"
fi
```

---

## Extending the Job

### Add More Parameters

1. **Build Version**:

   - Type: String Parameter
   - Name: `VERSION`
   - Default: `1.0.0`

2. **Enable Debug Mode**:

   - Type: Boolean Parameter
   - Name: `DEBUG`
   - Default: Unchecked

3. **Target Region**:
   - Type: Choice Parameter
   - Name: `REGION`
   - Choices: `us-east-1`, `us-west-2`, `eu-west-1`

Updated script:

```bash
#!/bin/bash

echo "Build Configuration:"
echo "==================="
echo "Stage: $Stage"
echo "Environment: $env"
echo "Version: ${VERSION:-1.0.0}"
echo "Debug Mode: ${DEBUG:-false}"
echo "Region: ${REGION:-us-east-1}"
echo "==================="

if [ "$DEBUG" = "true" ]; then
    set -x  # Enable debug mode
fi
```

---

## Pipeline Equivalent

For reference, here's how this would look as a Jenkins Pipeline:

```groovy
pipeline {
    agent any

    parameters {
        string(
            name: 'Stage',
            defaultValue: 'Build',
            description: 'Stage of the build process'
        )
        choice(
            name: 'env',
            choices: ['Development', 'Staging', 'Production'],
            description: 'Target environment for deployment'
        )
    }

    stages {
        stage('Display Parameters') {
            steps {
                script {
                    echo "Stage: ${params.Stage}"
                    echo "Environment: ${params.env}"
                }
            }
        }
    }
}
```

---

## Key Takeaways

1. ✅ Parameterized builds make Jenkins jobs flexible and reusable
2. ✅ String parameters allow free-text input with default values
3. ✅ Choice parameters provide a controlled set of options
4. ✅ Parameters are available as environment variables in shell scripts
5. ✅ "Build with Parameters" replaces "Build Now" when parameters are defined
6. ✅ Parameter validation in scripts ensures data integrity
7. ✅ Different parameter types serve different use cases
8. ✅ Proper naming and descriptions improve usability

---

## Validation Checklist

- ✅ Job `parameterized-job` created successfully
- ✅ String parameter `Stage` with default value `Build` configured
- ✅ Choice parameter `env` with three options configured
- ✅ Shell command echoes both parameter values
- ✅ Build executed successfully with `Production` environment
- ✅ Console output displays correct parameter values
- ✅ "Build with Parameters" button is visible
- ✅ Job can be run multiple times with different parameters

---

## Additional Resources

- [Jenkins Parameterized Build Documentation](https://www.jenkins.io/doc/book/pipeline/syntax/#parameters)
- [Jenkins Parameter Types Reference](https://plugins.jenkins.io/parameter-separator/)
- [Best Practices for Parameterized Builds](https://www.jenkins.io/doc/book/pipeline/syntax/#parameters-directive)
- [Extended Choice Parameter Plugin](https://plugins.jenkins.io/extended-choice-parameter/)

---

**Task Completed Successfully! ✓**

The new DevOps Engineer now understands how to create and use parameterized Jenkins jobs, which will serve as a foundation for more complex automation tasks in the future. This simple example demonstrates the power of making Jenkins jobs flexible and reusable through parameters.
