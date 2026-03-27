## ADDED Requirements

### Requirement: Chinese translation file naming convention
The system SHALL create translation files with the naming pattern `{original-filename}.zh-CN.md`.

#### Scenario: Creating translation file for README
- **WHEN** translating README.md
- **THEN** the translation file MUST be named README.zh-CN.md

#### Scenario: Creating translation file for nested path
- **WHEN** translating docs/guide/getting-started.md
- **THEN** the translation file MUST be docs/guide/getting-started.zh-CN.md
