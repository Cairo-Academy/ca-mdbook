# Enhanced Contribution Guidelines

Thank you for your interest in contributing to our project! These guidelines will walk you through the entire contribution process with detailed explanations to make your experience smooth and successful.

## Prerequisites

Before contributing, you'll need:

- A GitHub account
- Basic familiarity with Git and Markdown
- [mdBook](https://rust-lang.github.io/mdBook/guide/installation.html) installed on your system

## Detailed Contribution Steps

### 1. Setting Up mdBook

**Why this is needed**: mdBook is the tool we use to build and serve our documentation website.

1. Follow the [official mdBook installation guide](https://rust-lang.github.io/mdBook/guide/installation.html)
2. Verify installation by running:
   ```bash
   mdbook --version
   ```
   
**Troubleshooting**:
- If you encounter permission errors, try installing with `cargo install mdbook --locked`
- Ensure you have Rust installed if using the cargo method

### 2. Forking and Cloning the Repository

**Why this is needed**: This creates your personal copy of the project where you can make changes.

1. Click the "Fork" button at the top right of the repository page
2. Clone your forked repository:
   ```bash
   git clone https://github.com/YOUR-USERNAME/REPOSITORY-NAME.git
   cd REPOSITORY-NAME
   ```

### 3. Understanding the Project Structure

The key files you'll work with:
All within the `/src` directory
- `SUMMARY.md`: The table of contents that organizes all chapters
- Individual chapter files (e.g., `contract/erc20.md`): The content files
- `book.toml`: Configuration for mdBook

### 4. Making Your Contribution

#### Adding a New Chapter

1. Create a new Markdown file in the appropriate directory (e.g., `contract/new_chapter.md`)
2. Add your content following the same format as existing chapters
3. Edit `SUMMARY.md` to include your new chapter in the table of contents

**Example**: To add a new ERC721 chapter under "Starknet Smart Contracts":
```markdown
- [ERC721 Token Contract](./contract/erc721.md)
```

#### Editing an Existing Chapter

1. Locate the corresponding Markdown file
2. Make your changes while preserving the existing structure
3. For major changes, consider creating a new section rather than replacing content

### 5. Previewing Your Changes

**Why this is needed**: This lets you verify your changes look correct before submitting.

1. Build the book:
   ```bash
   mdbook build
   ```
2. Serve the book locally:
   ```bash
   mdbook serve --open
   ```
   This will automatically open your browser to `http://localhost:3000`

**Common Issues**:
- If links don't work, check your `summary.md` entries
- If formatting looks wrong, validate your Markdown syntax

### 6. Submitting Your Changes

1. Stage your changes:
   ```bash
   git add .
   ```
2. Commit with a descriptive message:
   ```bash
   git commit -m "Add new chapter on ERC721 contracts"
   ```
3. Push to your fork:
   ```bash
   git push origin main
   ```
4. Create a pull request from your fork to the main repository

## Best Practices

1. **Keep chapters focused**: Each chapter should cover one clear topic
2. **Use consistent formatting**: Follow the style of existing chapters
3. **Add examples**: Where applicable, include code samples like the ERC20 example
4. **Explain concepts**: Assume readers are familiar with blockchain basics but new to StarkNet
5. **Reference sources**: When drawing from external materials, provide attribution

## Troubleshooting

- **Build errors**: Check for syntax errors in your Markdown
- **Missing chapters**: Verify your `summary.md` entry points to the correct file
- **Merge conflicts**: Sync your fork with the main repository regularly

## Need Help?

If you encounter any issues or have questions:
- Open an issue in the repository
- [Join our community chat](https://t.me/+uQKuqWrTlhs5ZWI0)

We appreciate your contribution and look forward to reviewing your pull request! 🎉