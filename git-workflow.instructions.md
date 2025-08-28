**Important**: Always ask for permission before performing any git operation if there are uncommitted changes.

This project follows a Git Flow based branching strategy to ensure code quality and collaboration. Follow these guidelines:

1. **Branch Naming Convention**
   - Use descriptive names following the pattern:
     - `feature/[feature_name]` - For new features (e.g., `feature/user-authentication`)
     - `bugfix/[issue_description]` - For bug fixes (e.g., `bugfix/login-validation-error`)
     - `release/[version]` - For release preparation (e.g., `release/1.2.0`)
     - `hotfix/[issue_description]` - For critical production fixes (e.g., `hotfix/payment-gateway-failure`)

2. **Commit Guidelines**
   - Write clear, descriptive commit messages
   - Use conventional commits format: `type(scope): description`
   - Available types:
     - `feat`: New features
     - `fix`: Bug fixes
     - `docs`: Documentation changes
     - `style`: Code style changes (formatting, etc.)
     - `refactor`: Code refactoring
     - `test`: Adding or updating tests
     - `chore`: Maintenance tasks
   - Use emojis from https://gitmoji.dev to categorize commits
   - Use the present tense in commit messages
   - Write commit description that would summarize the changes concisely in bullets
   - Example:
     ```text
     ✨ feat(auth): add user authentication feature
     
     - Implement login and registration components
     - Add API integration for user authentication
     - Store user session in localStorage
     - Add route guards for protected pages
     ```
   - Keep commits focused and atomic
   - Run pre-commit hooks and resolve any errors before committing

3. **Pre-commit Checklist**
   - Run `npm run lint` and fix any linting errors
   - Run `npm run type-check` to ensure TypeScript compilation
   - Run `npm run test` to ensure all tests pass
   - Run `npm run build` to verify the build works
   - Format code with `npm run format`

4. **Pull Request Guidelines**
   - Use descriptive PR titles following conventional commits format
   - Include a detailed description of changes
   - Link related issues using keywords (fixes #123, closes #456)
   - Request reviews from relevant team members
   - Ensure CI/CD pipeline passes
   - Update documentation if needed
   - Add screenshots for UI changes

5. **Code Review Standards**
   - Review for code quality, performance, and security
   - Check for proper TypeScript usage and type safety
   - Verify Vue.js best practices are followed
   - Ensure accessibility guidelines are met
   - Check for proper error handling
   - Verify test coverage for new features

6. **Release Process**
   - Create release branch from develop
   - Update version in `package.json`
   - Update CHANGELOG.md with new features and fixes
   - Test thoroughly in staging environment
   - Merge to main branch and tag the release
   - Deploy to production
   - Merge back to develop branch

7. **Hotfix Process**
   - Create hotfix branch from main
   - Implement minimal fix
   - Test thoroughly
   - Merge to both main and develop
   - Tag as patch release
   - Deploy immediately

8. **Git Hooks Setup**
   - Install husky for git hooks: `npm run prepare`
   - Pre-commit hooks run linting and formatting
   - Pre-push hooks run tests
   - Configure hooks to prevent bad commits

Example `.husky/pre-commit`:
```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run lint-staged
npm run type-check
```

Example `.husky/pre-push`:
```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run test
npm run build
```

9. **Branch Protection Rules**
   - Require PR reviews before merging to main/develop
   - Require status checks to pass
   - Require branches to be up to date before merging
   - Restrict force pushes to protected branches
   - Require signed commits for enhanced security
