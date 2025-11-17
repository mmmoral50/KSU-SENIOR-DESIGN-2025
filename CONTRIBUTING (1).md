# Contributing to PlantGuard IoT

Thank you for your interest in contributing to PlantGuard! This is a senior project for King Saud University (KSU), and we welcome contributions from the community.

## 🚀 How to Contribute

### Reporting Bugs

If you find a bug, please create an issue with:
- Clear description of the problem
- Steps to reproduce
- Expected vs actual behavior
- Screenshots (if applicable)
- Your environment (OS, browser, ESP32 model)

### Suggesting Features

We love feature suggestions! Please create an issue with:
- Clear description of the feature
- Use cases and benefits
- Any implementation ideas

### Pull Requests

1. **Fork the repository**
   ```bash
   git clone https://github.com/yourusername/plantguard-iot.git
   ```

2. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes**
   - Follow the existing code style
   - Add comments where needed
   - Test your changes thoroughly

4. **Commit your changes**
   ```bash
   git commit -m "Add: Brief description of your changes"
   ```
   
   Use conventional commits:
   - `Add:` for new features
   - `Fix:` for bug fixes
   - `Update:` for improvements
   - `Docs:` for documentation changes

5. **Push to your fork**
   ```bash
   git push origin feature/your-feature-name
   ```

6. **Create a Pull Request**
   - Describe your changes clearly
   - Reference any related issues
   - Wait for review and feedback

## 💻 Development Setup

### Prerequisites
- Node.js 18+
- PostgreSQL database
- Arduino IDE (for ESP32 firmware)
- ESP32 board (Heltec WiFi LoRa 32 V3.2 recommended)

### Local Development

1. **Install dependencies**
   ```bash
   npm install
   ```

2. **Set up environment**
   ```bash
   cp .env.example .env
   # Edit .env with your credentials
   ```

3. **Initialize database**
   ```bash
   npm run db:push
   ```

4. **Start development server**
   ```bash
   npm run dev
   ```

## 🏗️ Project Structure

```
plantguard-iot/
├── client/          # React frontend
│   └── src/
│       ├── pages/   # Page components
│       ├── components/ # Reusable components
│       └── lib/     # Utilities
├── server/          # Express backend
│   ├── routes.ts    # API endpoints
│   ├── storage.ts   # Database layer
│   └── mqtt-broker.ts # MQTT integration
├── shared/          # Shared types/schemas
├── *.ino            # ESP32 firmware files
└── docs/            # Documentation
```

## 🎨 Code Style

### TypeScript/JavaScript
- Use TypeScript for type safety
- Follow existing ESLint rules
- Use async/await over callbacks
- Add JSDoc comments for complex functions

### React Components
- Use functional components with hooks
- Keep components small and focused
- Use descriptive prop names
- Add data-testid for testing

### Database
- Use Drizzle ORM
- Define schemas in `shared/schema.ts`
- Add migrations when needed

### ESP32 Firmware
- Follow Arduino style guidelines
- Add comments for complex logic
- Test on hardware before submitting

## 🧪 Testing

Before submitting a PR:
- Test the web dashboard
- Test API endpoints
- Test ESP32 firmware (if modified)
- Verify mobile PWA still works
- Check console for errors

## 📝 Documentation

When adding features:
- Update README.md if needed
- Add comments to complex code
- Update relevant .md files
- Include setup instructions

## 🔒 Security

- Never commit `.env` files
- Don't include API keys or passwords
- Use environment variables for secrets
- Report security issues privately

## 🤝 Code of Conduct

- Be respectful and professional
- Help others learn and grow
- Accept constructive feedback
- Focus on what's best for the project

## 📞 Questions?

- Open an issue for general questions
- Tag maintainers for urgent issues
- Check existing documentation first

## 🙏 Thank You!

Every contribution helps make PlantGuard better for everyone. Whether it's code, documentation, or bug reports - we appreciate your help!

---

**Happy Contributing! 🌱**
