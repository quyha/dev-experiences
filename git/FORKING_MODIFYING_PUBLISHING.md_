## Forking, Modifying, and Publishing NPM Packages — For those almost-perfect packages

### Good to know beforehand

You’ll need to sign up for an npm account.
```bash
$ npm login
```

### Setup

1. Fork the repo and clone it to your computer.
2. Make the changes you want to the package’s source files.
3. Update the package.json name (must be unique on npm) and version number. If this is a minor fork mostly for yourself, I’d recommend using the naming convention ‘@yourUsername/pkgName’.
4. Commit your changes.
5. In your terminal npm login with your username and password. You’ll also confirm your public email address.
6. Run npm publish --access public. The ‘–access public’ amendment is only needed if you use the ‘@yourUsername/pkgName’, and if it’s the package’s first publication. Subsequent updates to the do not require the flag.

### Updates

1. Update the package.json version number.
2. [Only if necessary] Build with npm run build or yarn build, then run npm update [package name] to update the package.
3. Else, run npm publish.

### Common Errors
1. Code: 402 You must sign up for private packages : @yourUsername/pkgName: You need to append --access public to the end of `npm publish.
