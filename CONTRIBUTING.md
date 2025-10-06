# Contributing to ansible-role-clickhouse

Thank you for your interest in contributing! This document provides guidelines for contributing to this project.

## How to Contribute

### Reporting Issues

- Check existing issues before creating a new one
- Provide clear description and steps to reproduce
- Include Ansible version, OS details, and error messages
- Use issue templates when available

### Pull Requests

1. **Fork the repository**

   ```bash
   git clone https://github.com/anhnt094/ansible-role-clickhouse.git
   cd ansible-role-clickhouse
   ```

2. **Create a feature branch**

   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes**

   - Follow existing code style
   - Update documentation if needed
   - Add tests for new features
   - Ensure all tests pass

4. **Test your changes**

   ```bash
   # Syntax check
   ansible-playbook tests/test.yml --syntax-check

   # Lint check
   yamllint .
   ansible-lint

   # Test deployment
   ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-standalone.yml
   ```

5. **Commit your changes**

   ```bash
   git add .
   git commit -m "feat: add your feature description"
   ```

6. **Push and create PR**
   ```bash
   git push origin feature/your-feature-name
   ```

### Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `style:` Code style changes (formatting)
- `refactor:` Code refactoring
- `test:` Adding or updating tests
- `chore:` Maintenance tasks

Examples:

```
feat: add support for Ubuntu 22.04
fix: correct IPv6 binding issue
docs: update inventory examples
```

## Development Guidelines

### Code Style

- Use 2 spaces for indentation (YAML)
- Keep lines under 160 characters
- Use meaningful variable names
- Add comments for complex logic
- Follow Ansible best practices

### Testing

- Test on Ubuntu 24.04 LTS
- Verify standalone and cluster deployments
- Test reset playbook thoroughly
- Check both XML and YAML config formats

### Documentation

- Update README.md for user-facing changes
- Update CHANGELOG.md following Keep a Changelog format
- Add examples for new features
- Keep documentation concise and clear

## Project Structure

```
ansible-role-clickhouse/
├── defaults/         # Default variables
├── handlers/         # Service handlers
├── meta/            # Role metadata
├── tasks/           # Main tasks
├── templates/       # Jinja2 templates
├── tests/           # Test playbooks
├── test-playbooks/  # Example playbooks (git-ignored)
└── vars/            # OS-specific variables
```

## Release Process

1. Update version in `meta/main.yml`
2. Update `CHANGELOG.md`
3. Create git tag: `git tag -a v1.0.0 -m "Release v1.0.0"`
4. Push tag: `git push origin v1.0.0`
5. Create GitHub release with changelog

## Questions?

- Open an issue for questions
- Check existing documentation first
- Be respectful and constructive

Thank you for contributing! 🎉
