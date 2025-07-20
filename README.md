# Bedrock Project - Vue.js Applications Collection

A collection of Vue.js applications demonstrating different use cases and AWS integrations. This repository contains two separate applications, each designed for independent deployment on AWS Amplify.

## 📱 Applications

### 1. Vue Demo App (`vue-amplify-app/`)
**Simple Counter Application**

A clean demonstration of Vue 3 fundamentals featuring an interactive counter.

- **Purpose**: Vue.js basics and Amplify deployment showcase
- **Features**: Interactive counter, clean UI, responsive design
- **Tech Stack**: Vue 3, Vite, Composition API
- **Deployment**: AWS Amplify ready

[📖 View Documentation](./vue-amplify-app/README.md) | [🚀 Deploy to Amplify](#deployment-guide)

### 2. Bedrock Chat (`chat/`)
**AI-Powered Chat Application**

A sophisticated chat application with AWS Bedrock integration for AI conversations.

- **Purpose**: AWS Bedrock AI integration demo
- **Features**: Multi-model AI chat, real-time messaging, beautiful UI
- **Tech Stack**: Vue 3, AWS SDK, Bedrock API, Component architecture
- **Deployment**: AWS Amplify with environment variables

[📖 View Documentation](./chat/README.md) | [🚀 Deploy to Amplify](#deployment-guide)

## 🏗️ Architecture

```
Bedrock-project/
├── vue-amplify-app/          # Simple Vue demo app
│   ├── src/
│   ├── amplify.yml           # Amplify config for demo app
│   ├── package.json
│   └── README.md
├── chat/                     # Bedrock Chat application
│   ├── src/
│   │   ├── components/       # Modular Vue components
│   │   └── ...
│   ├── amplify.yml           # Amplify config for chat app
│   ├── package.json
│   └── README.md
└── README.md                 # This file
```

## 🚀 Quick Start

### Prerequisites

- **Node.js** 20 or higher
- **npm** or yarn package manager
- **AWS Account** (for deployment)
- **GitHub Account** (for repository connection)

### Local Development

Choose the application you want to work with:

#### Vue Demo App
```bash
cd vue-amplify-app
npm install
npm run dev
```

#### Bedrock Chat
```bash
cd chat
npm install
npm run dev
```

## 🌐 Deployment Guide

Both applications are configured for independent AWS Amplify deployment. Each has its own `amplify.yml` configuration optimized for its specific requirements.

### Option 1: Amplify Console (Recommended)

#### Deploy Vue Demo App

1. **Create New Amplify App**
   - Go to [AWS Amplify Console](https://console.aws.amazon.com/amplify/)
   - Click "New app" → "Host web app"
   - Connect your GitHub repository

2. **Configure Build Settings**
   - **Repository**: Select this repository
   - **Root Directory**: `vue-amplify-app`
   - **Build Settings**: Amplify will auto-detect `amplify.yml`

3. **Deploy**
   - Click "Save and deploy"
   - App will be available at generated URL

#### Deploy Bedrock Chat

1. **Create Second Amplify App**
   - Repeat the process for a new app
   - **Root Directory**: `chat`
   - **Environment Variables** (Optional):
     ```
     VITE_AWS_REGION=us-east-1
     VITE_AWS_ACCESS_KEY_ID=your-access-key
     VITE_AWS_SECRET_ACCESS_KEY=your-secret-key
     ```

2. **Configure AWS Bedrock**
   - Enable desired models in AWS Bedrock console
   - Update credentials in the app configuration

### Option 2: Amplify CLI

#### Setup (One-time)
```bash
# Install Amplify CLI
npm install -g @aws-amplify/cli

# Configure with your AWS credentials
amplify configure
```

#### Deploy Demo App
```bash
cd vue-amplify-app
amplify init
amplify add hosting
amplify publish
```

#### Deploy Chat App
```bash
cd chat
amplify init
amplify add hosting
amplify publish
```

## 🔧 Configuration

### Environment Variables

#### Chat App (Bedrock Integration)
```bash
# Required for AWS Bedrock
VITE_AWS_REGION=us-east-1
VITE_AWS_ACCESS_KEY_ID=your-access-key
VITE_AWS_SECRET_ACCESS_KEY=your-secret-key

# Optional: Custom endpoint
VITE_BEDROCK_ENDPOINT=https://bedrock-runtime.us-east-1.amazonaws.com
```

### AWS Bedrock Setup

1. **Enable Models**
   - Go to AWS Bedrock Console
   - Request access to desired models:
     - Claude 3.5 Sonnet
     - Claude 3 Haiku
     - Amazon Titan Text Premier

2. **IAM Permissions**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "bedrock:InvokeModel",
           "bedrock:ListFoundationModels"
         ],
         "Resource": "*"
       }
     ]
   }
   ```

## 🛠️ Development

### Project Structure
- **Independent Applications**: Each app has its own dependencies and build process
- **Shared Patterns**: Both use Vue 3, Vite, and similar project structure
- **Separate Deployments**: Each app deploys independently with its own configuration

### Adding New Features

#### To Demo App
- Simple UI components and interactions
- Focus on Vue.js fundamentals
- Keep dependencies minimal

#### To Chat App
- Advanced AWS integrations
- Complex component architecture
- Enhanced UI/UX features

## 🔒 Security

Both applications include production-ready security configurations:

- **HTTPS Enforcement**: Strict Transport Security headers
- **Content Security Policy**: Prevents XSS attacks
- **Secure Headers**: X-Frame-Options, X-Content-Type-Options
- **Optimized Caching**: Performance without security compromise

## 📊 Monitoring

### Amplify Console Features
- **Build Logs**: Monitor deployment process
- **Performance Metrics**: Track app performance
- **Custom Domains**: Configure custom URLs
- **Branch Deployments**: Multiple environment support

### AWS CloudWatch
- Application logs and metrics
- Error tracking and alerting
- Performance monitoring

## 🤝 Contributing

1. **Fork the repository**
2. **Choose your app**: Work in either `vue-amplify-app/` or `chat/`
3. **Create feature branch**: `git checkout -b feature/app-name/amazing-feature`
4. **Test locally**: Ensure your changes work in development
5. **Submit PR**: Include details about which app was modified

### Code Standards
- **Vue 3 Composition API**: Use modern Vue patterns
- **Component Architecture**: Keep components focused and reusable
- **Security First**: Follow security best practices
- **Documentation**: Update README files for significant changes

## 📄 License

This project is licensed under the MIT License - see individual app directories for details.

## 🆘 Support

- **Issues**: Create GitHub issues for bugs or feature requests
- **Documentation**: Check individual app README files
- **AWS Support**: Consult AWS Amplify and Bedrock documentation

## 🔗 Links

- [AWS Amplify Documentation](https://docs.amplify.aws/)
- [AWS Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- [Vue.js Documentation](https://vuejs.org/)
- [Vite Documentation](https://vitejs.dev/)

---

**Built with ❤️ using Vue.js, AWS Amplify, and AWS Bedrock**