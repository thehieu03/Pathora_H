## ADDED Requirements

### Requirement: Vercel production build excludes dev-only packages

The Vercel CI/CD pipeline SHALL install only production dependencies, excluding any packages in `devDependencies` of the root `package.json` at the deploy target directory.

#### Scenario: npm install on Vercel production build
- **WHEN** Vercel triggers a production build for `pathora/`
- **THEN** the `npm install` command SHALL be executed with `--omit=dev` flag
- **AND** packages such as `gitnexus`, `tree-sitter-cli`, or any other dev-only tools SHALL NOT be present in the resulting `node_modules`

#### Scenario: gitnexus removed from root dependencies
- **WHEN** the `pathora/package.json` is read
- **THEN** the `dependencies` field SHALL NOT contain `gitnexus`
- **AND** the `devDependencies` field SHALL NOT contain `gitnexus`
