# 🚀 CI/CD Pipeline for Flask Application using GitHub Actions 🚀

This project sets up a Continuous Integration and Continuous Deployment (CI/CD) pipeline for a Flask application using GitHub Actions. The pipeline automatically tests, merges, and deploys code to an EC2 instance whenever changes are pushed to the `dev` branch.

## 📁 Project Structure

```
.
├── .github
│   └── workflows
│       └── deploy.yml         # GitHub Actions workflow for testing and deploying
├── app.py                      # Flask application code
└── tests.py                    # Test cases for the Flask application
```

## 🔧 Workflow Overview

### 1. Testing

When changes are pushed to the `dev` branch, the following steps occur:
- **Checkout Code**: The code from the `dev` branch is checked out.
- **Set Up Python**: The Python environment is set up using the specified version.
- **Install Dependencies**: Required dependencies, including Flask, Pytest, and Requests, are installed.
- **Run Tests**: The test cases are executed against the Flask application.

### 2. Merging

If all tests pass:
- The code is merged into the `main` branch using the `GITHUB_TOKEN` for authentication.

### 3. Deployment

After merging:
- The application is deployed to an AWS EC2 instance with the following steps:
  - **Checkout Main Branch**: The code from the `main` branch is checked out.
  - **Deploy to EC2**: The application code is copied to the EC2 instance, where the environment is set up and the Flask application is started using Gunicorn.

## 🌐 GitHub Actions Workflow

The GitHub Actions workflow is defined in `.github/workflows/deploy.yml`. Here’s a brief look at its configuration:

```yaml
name: Test, Merge, and Deploy

on:
  push:
    branches:
      - dev

jobs:
  test_and_merge:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3
        with:
          ref: dev
          fetch-depth: 0  # Fetch all branches to allow merging

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Install Dependencies
        run: |
          python -m pip install --upgrade pip
          pip install flask pytest requests  # Install dependencies for app and testing

      - name: Start Flask Application
        run: |
          nohup python app.py &
          sleep 5  # Give the app time to start

      - name: Run Tests
        run: |
          pytest tests.py

      - name: Merge to Main if Tests Pass
        if: success()  # Proceed only if tests passed
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          git config --global user.name "GitHub Actions"
          git config --global user.email "actions@github.com"
          git checkout main
          git merge dev
          git push origin main

  deploy_to_ec2:
    runs-on: ubuntu-latest
    needs: test_and_merge  # Run only if test_and_merge job succeeds

    steps:
      - name: Checkout Main Branch
        uses: actions/checkout@v3
        with:
          ref: main  # Ensure we're deploying the main branch code
          fetch-depth: 0  # Fetch all branches to ensure we have the latest changes

      - name: Deploy to EC2
        env:
          EC2_HOST: ${{ secrets.EC2_HOST }}
          EC2_USER: ${{ secrets.EC2_USER }}
        run: |
          echo "${{ secrets.EC2_SSH_KEY }}" > ec2_key.pem
          chmod 600 ec2_key.pem
          
          # Copy application code to EC2
          scp -i ec2_key.pem -o StrictHostKeyChecking=no app.py $EC2_USER@$EC2_HOST:/home/$EC2_USER/app.py
          
          # SSH into EC2 to set up environment and run the app
          ssh -i ec2_key.pem -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST << 'EOF'
            sudo apt update
            sudo apt install -y python3 python3-pip
            
            # Install Flask and Gunicorn
            pip3 install --user flask gunicorn
            
            # Kill any existing app process and start app with Gunicorn
            pkill -f gunicorn || true
            nohup gunicorn -w 4 -b 0.0.0.0:5000 app:app --daemon &
          EOF
```

## 📦 Prerequisites

- **AWS EC2 Instance**: Ensure you have an EC2 instance running, and SSH access is configured.
- **GitHub Repository**: This project should be hosted on GitHub.

## 🔑 Secrets Configuration

Add the following secrets to your GitHub repository:
- `EC2_HOST`: The public IP or hostname of your EC2 instance.
- `EC2_USER`: The username for SSH access (e.g., `ec2-user`).
- `EC2_SSH_KEY`: The private SSH key used to access the EC2 instance.
- `GITHUB_TOKEN`: Automatically provided by GitHub Actions, used for merging branches.

## ⚙️ Getting Started

1. Clone the repository:

   ```bash
   git clone https://github.com/devops-bharat05/GithubAction_myapp.git
   ```

2. Navigate to the project directory:

   ```bash
   cd GithubAction_myapp
   ```

3. Make changes to `app.py` or `tests.py`, then push to the `dev` branch:

   ```bash
   git checkout dev
   git add .
   git commit -m "Update app and tests"
   git push origin dev
   ```

4. Monitor the GitHub Actions tab in your repository to see the workflow in action!
---

> 💡 **Tip**: Regularly monitor your CI/CD pipeline and logs to ensure everything runs smoothly.
```

### Icons Used
- 🚀 for project title
- 📁 for project structure
- 🔧 for workflow overview
- 🌐 for GitHub Actions workflow
- 📦 for prerequisites
- 🔑 for secrets configuration
- ⚙️ for getting started
- 🎉 for conclusion
- 💡 for tips

Feel free to customize the content further to match your project's specifics or to add more details as needed!
---
## 🎉 Conclusion

This setup allows for automated testing and deployment of your Flask application, streamlining your development process. Enjoy coding and deploying with confidence!
