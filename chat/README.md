# Bedrock Chat - Vue 3 Chat Application

A modern chat application built with Vue 3 and designed to integrate with AWS Bedrock for AI-powered conversations.

## ✨ Features

- 🤖 **AWS Bedrock Integration** - Connect to multiple AI models (Claude, Titan, etc.)
- 🎨 **Beautiful UI** - Modern gradient design with smooth animations
- 📱 **Responsive Design** - Works on desktop and mobile devices
- 🔄 **Real-time Status** - Connection status indicator
- 💬 **Rich Messaging** - Support for markdown formatting (bold, italic)
- 🎯 **Model Selection** - Choose between different AI models
- ⚡ **Fast Performance** - Built with Vite for optimal build times

## 🚀 Quick Start

### Prerequisites

- Node.js 20 or higher
- npm or yarn package manager

### Local Development

1. **Clone the repository**
   ```bash
   git clone https://github.com/matthew-thomasson/Bedrock-project.git
   cd Bedrock-project/chat
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Start development server**
   ```bash
   npm run dev
   ```

4. **Open your browser**
   Navigate to `http://localhost:5173`

### Build for Production

```bash
npm run build
```

The built files will be in the `dist/` directory.

## 🌐 AWS Amplify Deployment

This app is pre-configured for AWS Amplify deployment with an optimized `amplify.yml` file.

### Deploy to Amplify

1. **Connect to GitHub**
   - Log into the [AWS Amplify Console](https://console.aws.amazon.com/amplify/)
   - Click "New app" → "Host web app"
   - Connect your GitHub repository

2. **Configure Build Settings**
   - Amplify will automatically detect the `amplify.yml` file
   - Set the root directory to `chat` if deploying from this subdirectory
   - The build configuration is already optimized for Vue 3 + Vite

3. **Environment Variables** (Optional)
   Add these if using AWS Bedrock:
   ```
   VITE_AWS_REGION=us-east-1
   VITE_AWS_ACCESS_KEY_ID=your-access-key
   VITE_AWS_SECRET_ACCESS_KEY=your-secret-key
   ```

4. **Deploy**
   - Click "Save and deploy"
   - Your app will be available at a generated URL

### Manual Amplify CLI Deployment

```bash
# Install Amplify CLI
npm install -g @aws-amplify/cli

# Configure Amplify
amplify configure

# Initialize Amplify in your project
amplify init

# Add hosting
amplify add hosting

# Publish your app
amplify publish
```

## 🔧 Configuration

### AWS Bedrock Setup

1. **Enable Models in AWS Console**
   - Go to AWS Bedrock console
   - Enable access to desired models (Claude, Titan, etc.)

2. **Update Credentials**
   - Uncomment the AWS SDK initialization code in `src/App.vue`
   - Add your AWS credentials via environment variables or IAM roles

### Customization

- **Styling**: Modify `src/style.css` for UI customization
- **Models**: Update model options in `src/components/ChatHeader.vue`
- **API Integration**: Modify the `callBedrock` function in `src/App.vue`

## 📁 Project Structure

```
chat/
├── public/
│   ├── _redirects          # SPA routing configuration
│   └── vite.svg           # App icon
├── src/
│   ├── components/
│   │   ├── ChatHeader.vue  # Header with model selector
│   │   ├── ChatInput.vue   # Message input component
│   │   └── ChatMessage.vue # Individual message display
│   ├── App.vue            # Main application component
│   ├── main.js            # Vue app initialization
│   └── style.css          # Global styles
├── amplify.yml            # Amplify build configuration
├── index.html             # HTML entry point
├── package.json           # Dependencies and scripts
├── vite.config.js         # Vite configuration
└── README.md              # This file
```

## 🛠️ Available Scripts

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build locally

## 🔒 Security Features

The `amplify.yml` configuration includes:

- **Security Headers**: HSTS, CSP, X-Frame-Options, etc.
- **Content Security Policy**: Restricts resource loading for security
- **Optimized Caching**: Long-term caching for static assets
- **AWS Integration**: Secure connection to AWS services

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License.

## 🆘 Support

For issues and questions:
- Create an issue on GitHub
- Check the AWS Amplify documentation
- Review the Vue 3 documentation

---

Built with ❤️ using Vue 3, Vite, and AWS Amplify