# OpenCode Interview Platform: Complete Guide

## Overview

OpenCode Interview Platform is a free, open-source real-time collaborative coding platform designed for technical interviews. It serves as a complete alternative to commercial platforms like CoderPad and HackerRank, offering similar features at zero cost.

## Key Features

### Core Capabilities
- **Real-time Collaborative Editing**: See changes instantly across all participants
- **Live Code Execution**: Run code directly in the browser
- **16+ Programming Languages**: Comprehensive language support
- **Role-based Access**: Separate permissions for interviewers and candidates
- **Simple Session Management**: 6-digit codes for easy sharing (like Zoom)
- **Live Presence Indicators**: See who's online and active
- **Multiple Themes**: Dark mode and other theme options
- **Zero Setup for Candidates**: No installation required

### Supported Languages
- JavaScript, Python, Java, C++, C#, PHP, Ruby, Go, Rust, Swift
- TypeScript, Kotlin, Scala, R, SQL, HTML/CSS, Shell
- And more through extensible configuration

## Architecture

### Technology Stack
- **Frontend**: HTML, CSS, JavaScript
- **Real-time Database**: Firebase Realtime Database
- **Collaboration Engine**: Firepad (Operational Transformation)
- **Code Editor**: ACE Editor (Industry standard)
- **Code Execution**: Piston API (Free service)
- **Hosting**: Firebase Hosting (Free tier)

### System Components

#### 1. Authentication System
- Simple role-based authentication
- Default admin credentials for interviewers
- Anonymous access for candidates

#### 2. Session Management
- 6-digit session codes
- Real-time participant tracking
- Session persistence

#### 3. Code Collaboration
- Operational transformation for conflict resolution
- Real-time cursor and selection sharing
- Live presence indicators

#### 4. Code Execution
- Browser-based code execution
- Multiple language support
- Input/output handling

## Cost Comparison

| Feature | OpenCode | CoderPad | HackerRank |
|---------|----------|----------|------------|
| Monthly Cost | **$0** | $599+ | $819+ |
| Real-time Collaboration | ✅ | ✅ | ✅ |
| Code Execution | ✅ | ✅ | ✅ |
| Multiple Languages | ✅ (16+) | ✅ | ✅ |
| Custom Branding | ✅ | ❌ | ❌ |
| Self-Hosted | ✅ | ❌ | ❌ |
| Open Source | ✅ | ❌ | ❌ |

## Setup Guide

### Quick Setup (2 minutes)

#### 1. Clone Repository
```bash
git clone https://github.com/humancto/opencode.git
cd opencode
```

#### 2. Set up Firebase
1. Go to [Firebase Console](https://console.firebase.google.com)
2. Create new project
3. Click ⚙️ Settings → "Your apps" → `</>` Web icon
4. Copy the configuration values

#### 3. Configure the App
```bash
# Copy example config
cp lib/firebase-sdk.example.js lib/firebase-sdk.js

# Edit lib/firebase-sdk.js and add your Firebase credentials
```

#### 4. Enable Realtime Database
1. In Firebase Console → Realtime Database
2. Create Database → Start in test mode

#### 5. Run the App
```bash
python3 -m http.server 8000
```

**Access at:** [http://localhost:8000](http://localhost:8000)

## Usage Guide

### For Interviewers

1. **Access Platform**: Navigate to the platform URL
2. **Login**: Click "I'm an Interviewer"
3. **Credentials**: Use default or custom admin credentials
   - Email: `admin@opencode.com`
   - Password: `admin123`
4. **Create Session**: Click "Create New Session"
5. **Share Code**: Provide the 6-digit code to candidates
6. **Monitor**: Watch candidate progress in real-time

### For Candidates

1. **Access Platform**: Navigate to the platform URL
2. **Join Session**: Click "I'm a Candidate"
3. **Enter Details**: Provide name and session code
4. **Start Coding**: Begin working on the interview problem
5. **Collaborate**: Interviewer can see code and provide guidance

## Customization

### Branding Customization

#### Change Platform Name
Edit `index.html`:
```html
<title>YourCompany - Coding Interviews</title>
<h2>YourCompany Code Editor</h2>
```

#### Change Admin Credentials
Edit `scripts/auth.js`:
```javascript
const ADMIN_CREDENTIALS = {
  email: 'your-email@company.com',
  password: 'your-secure-password'
};
```

### Language Customization

#### Add New Languages
Edit `scripts/firepad.js` and add to `languageConfig`:
```javascript
const languageConfig = {
  // ... existing languages
  scala: { mode: 'ace/mode/scala', ext: 'scala' },
  haskell: { mode: 'ace/mode/haskell', ext: 'hs' }
  // Add more as needed
};
```

#### Theme Customization
- All ACE editor themes are supported
- Add theme options in the theme selector dropdown in `index.html`

## Security Considerations

### Production Database Rules
For production deployment, update Firebase Realtime Database rules:
```json
{
  "rules": {
    "sessions": {
      "$sessionId": {
        ".read": true,
        ".write": true,
        ".validate": "newData.hasChildren(['firepad'])"
      }
    }
  }
}
```

### Best Practices
1. **Environment Variables**: Use for sensitive data in production
2. **Proper Authentication**: Consider Firebase Auth for production
3. **Rate Limiting**: Implement to prevent abuse
4. **Monitoring**: Track usage to stay within free tier limits

## Firebase Free Tier Limits

- **Realtime Database**: 1GB storage, 10GB/month transfer
- **Hosting**: 10GB storage, 360MB/day transfer
- **Simultaneous connections**: 100

### Capacity Estimates
- ~100 concurrent interviews
- ~1000 interviews per month
- ~10,000 code executions

## Troubleshooting

### Common Issues

#### "Permission Denied" Error
- Check Firebase Realtime Database rules
- Ensure rules allow read/write access
- Verify Firebase project configuration

#### Real-time Sync Not Working
- Check browser console for errors
- Verify Firebase credentials are correct
- Ensure HTTPS is used in production

#### Candidates Can't Type
- Clear browser cache
- Check for JavaScript errors
- Ensure only one tab per session is open

#### Code Execution Issues
- Verify Piston API availability
- Check language configuration
- Review execution timeout settings

## Advanced Features

### Integration Possibilities

#### Video/Audio Calling
- Integrate with WebRTC for video interviews
- Add screen sharing capabilities
- Implement recording features

#### Question Bank
- Create reusable interview questions
- Organize by difficulty and topic
- Track question usage and effectiveness

#### Automated Evaluation
- Implement code quality scoring
- Add test case validation
- Create performance metrics

### Enterprise Features

#### White-label Solution
- Custom branding and domain
- Company-specific configurations
- Branded candidate experience

#### Analytics Dashboard
- Interview performance metrics
- Candidate assessment data
- Usage statistics and reporting

#### Multi-tenant Support
- Separate organizations
- Role-based access control
- Custom configurations per tenant

## Deployment Options

### Self-Hosted Deployment

#### Firebase Hosting
- Free tier available
- Easy deployment process
- Automatic SSL certificates

#### Alternative Hosting
- Any static hosting service
- Custom domain configuration
- CDN integration for performance

### Enterprise Deployment

#### Private Cloud
- On-premises deployment
- Custom infrastructure
- Enhanced security controls

#### Hybrid Approach
- Mix of cloud and on-premises
- Flexible scaling options
- Custom integration points

## Contributing

### Development Setup
1. Fork the repository
2. Create feature branch
3. Make changes and test
4. Submit pull request

### Feature Development
- Follow existing code patterns
- Add comprehensive documentation
- Include tests where applicable
- Consider backward compatibility

### Community Guidelines
- Respectful communication
- Clear issue reporting
- Constructive feedback
- Collaborative problem-solving

## Roadmap

### Planned Features
- [ ] Video/audio calling integration
- [ ] Question bank system
- [ ] Automated testing/evaluation
- [ ] Multiple files support
- [ ] Whiteboard feature
- [ ] Recording & playback
- [ ] Analytics dashboard
- [ ] Mobile app support

### Enhancement Areas
- **Performance**: Optimize real-time collaboration
- **Security**: Enhanced authentication and authorization
- **UX/UI**: Improved user experience
- **Accessibility**: Better accessibility support
- **Internationalization**: Multi-language support

## Monetization Opportunities

While OpenCode is free, potential revenue streams include:

1. **Hosted SaaS Version**: $49/month for managed hosting
2. **Enterprise Support**: Custom development and support services
3. **Premium Features**: Advanced analytics, AI evaluation, etc.
4. **White-label Solutions**: Custom implementations for companies

## License

MIT License - Free for commercial and personal use

## Support and Resources

- **GitHub Repository**: https://github.com/humancto/OpenCode
- **Documentation**: Comprehensive README and setup guides
- **Community**: GitHub issues and discussions
- **Contributions**: Welcome from the open source community

## Acknowledgments

- **Firebase**: Real-time database and hosting
- **ACE Editor**: Industry-standard code editor
- **Piston**: Free code execution service
- **Firepad**: Operational transformation for collaboration

---

*OpenCode Interview Platform democratizes technical interviews by providing enterprise-grade features at zero cost. Perfect for startups, educational institutions, and companies looking to reduce hiring expenses while maintaining high-quality technical assessments.*