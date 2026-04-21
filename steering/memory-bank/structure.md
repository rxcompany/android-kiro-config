# Project Structure

## Module Organization

### App Module
- **app/**: 앱 진입점, DI 설정

### Domain Layer
- **domain/**: 비즈니스 로직, 도메인 모델

### Feature Modules
- bookmark, contents, dev, goods, live, main, review, scanner, schedule, search, share, shortform, showroom, user

### Library Modules
- analytics, chatting, design, eventbus, media, network, test, viewmodels, webview
- **foundation/**: Core utilities and extensions
- **message/**: Firebase Cloud Messaging and Android Notification infrastructure
- **navigation/**: References for navigation (e.g., arguments, common key)
- **share/**: Sharing utilities
- **user/**: User management and authentication

## Dependency Flow
```
app → features → domain
       ↓
    libraries
```
