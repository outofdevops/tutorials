## Overview of ECR

**Amazon Elastic Container Registry (ECR)** is a fully managed container image registry that makes it easy to store, manage, and deploy container images. Typically, the workflow involves:
1. Creating a repository in ECR.
2. Building a Docker image locally (or in a CI pipeline).
3. Authenticating Docker to your ECR registry.
4. Pushing the image to your ECR repository.
5. Pulling the image from ECR wherever you need to run it (e.g., ECS, EKS, or other platforms).

![Alt text](../.document360/assets/output.png?raw=true "Diagram")

---

## Prerequisites

1. **AWS CLI installed**  
   Make sure that the machine (CI runner, local dev machine, etc.) has the [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) installed.

2. **AWS CLI configured**  
   Configure AWS CLI with your credentials and default region. For example:
   ```bash
   aws configure
   ```
   You will be prompted to enter your AWS Access Key, Secret Key, default region, and output format (json is typical).

3. **Appropriate IAM permissions**  
   - `ecr:CreateRepository`
   - `ecr:DescribeRepositories`
   - (Optionally) `ecr:DeleteRepository`
   - And any other ECR permissions needed by your workflows.
   
   Since you mentioned that permissions are already set, you should be good to go.

---

## Step-by-Step: Creating an ECR Repository

### 1. Choose a Repository Name

Decide on a name for your repository. For example, let’s call it `my-awesome-app`.  
ECR repository names can include namespaces as well, such as `team/my-awesome-app`, to keep your images organized.

### 2. Create the Repository

Use the `aws ecr create-repository` command to create a repository in the default or specified AWS Region.

```bash
aws ecr create-repository \
  --repository-name my-awesome-app \
  --region <your-region>
```

Replace `<your-region>` with the AWS region where you want to create the repository, such as `us-east-1` or `eu-west-1`. You can omit `--region` if your `aws configure` default region is already set to the one you want.

**Example Output:**
```json
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:us-east-1:123456789012:repository/my-awesome-app",
        "registryId": "123456789012",
        "repositoryName": "my-awesome-app",
        "repositoryUri": "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-awesome-app",
        "createdAt": "2025-01-30T12:34:56-05:00",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}
```

### 3. Configure Optional Settings

Amazon ECR allows extra configuration:
- **Image scanning**: Enables automatic scanning of images on push.  
  ```bash
  aws ecr create-repository \
    --repository-name my-awesome-app \
    --image-scanning-configuration scanOnPush=true
  ```
- **Image tag mutability**: Choose whether tags are mutable or immutable.  
  ```bash
  aws ecr create-repository \
    --repository-name my-awesome-app \
    --image-tag-mutability IMMUTABLE
  ```

### 4. Verify the Repository

To verify creation, list the repositories:
```bash
aws ecr describe-repositories
```
You should see an entry for `my-awesome-app` in the output. This confirms the repository now exists.

---

## (Optional) Authentication and Pushing an Image

While your question focuses on creating repositories, the typical next steps involve pushing a container image. If you need to include those details, here they are:

1. **Authenticate Docker to ECR**  
   Use the AWS CLI to retrieve and execute a Docker login command:
   ```bash
   aws ecr get-login-password --region <your-region> \
     | docker login \
       --username AWS \
       --password-stdin 123456789012.dkr.ecr.<your-region>.amazonaws.com
   ```
   Replace `123456789012` with your AWS account ID and `<your-region>` with the correct region.

2. **Build or tag your Docker image**  
   If you have an existing Docker image named `my-awesome-app:latest`, tag it with your ECR repository URI:
   ```bash
   docker tag my-awesome-app:latest 123456789012.dkr.ecr.<your-region>.amazonaws.com/my-awesome-app:latest
   ```

3. **Push the image**  
   ```bash
   docker push 123456789012.dkr.ecr.<your-region>.amazonaws.com/my-awesome-app:latest
   ```

Once pushed, the image is stored in your new ECR repository and can be used across your AWS infrastructure or anywhere else you can pull Docker images.

---

## Best Practices & Tips

1. **Repository Naming Conventions**  
   - Consider using teams or application groups in your repository name (e.g., `teamA/my-app`) to help maintain a clear structure.
   - Use semantic versioning tags like `v1.0.0` alongside `latest` for more reliable rollbacks.

2. **Image Tag Mutability**  
   - **Immutable tags** help ensure that images with the same tag aren’t overwritten. This can be useful for production environments to avoid unexpected deployments.
   - **Mutable tags** are simpler but can lead to confusion if the same tag is re-pushed with different content.

3. **Scanning on Push**  
   - Enable **scanOnPush** to let Amazon ECR automatically scan images for vulnerabilities. This is useful in CI pipelines to ensure you catch issues early.

4. **CI/CD Integration**  
   - Automate these CLI steps in your CI pipeline to create repositories on-the-fly or to ensure they exist.
   - Tag images based on build IDs or commit hashes to maintain a clear version history.

5. **Lifecycle Policies**  
   - Implement lifecycle policies to automatically remove unneeded images from the repository after a period of time. This helps control storage costs.

6. **CloudTrail Logging**  
   - If you need an audit trail, ensure that AWS CloudTrail is set up to log ECR API calls (e.g., `CreateRepository`, `DeleteRepository`).

---

## Cleanup (If Needed)

To remove a repository (and all images inside it), use:
```bash
aws ecr delete-repository \
  --repository-name my-awesome-app \
  --force
```
The `--force` flag is required to delete a repository even if it contains images.

---

## Conclusion

With these steps, you can create Amazon ECR repositories via the AWS CLI and manage container images as part of your CI/CD process. Remember to always confirm you have the right AWS region, account IDs, and permissions.  

**Happy containerizing!**
