[![bugsplat-github-banner-basic-outline](https://user-images.githubusercontent.com/20464226/149019306-3186103c-5315-4dad-a499-4fd1df408475.png)](https://bugsplat.com)
<br/>
# <div align="center">BugSplat</div> 
### **<div align="center">Crash and error reporting built for busy developers.</div>**
<div align="center">
    <a href="https://twitter.com/BugSplatCo">
        <img alt="Follow @bugsplatco on Twitter" src="https://img.shields.io/twitter/follow/bugsplatco?label=Follow%20BugSplat&style=social">
    </a>
    <a href="https://discord.gg/K4KjjRV5ve">
        <img alt="Join BugSplat on Discord" src="https://img.shields.io/discord/664965194799251487?label=Join%20Discord&logo=Discord&style=social">
    </a>
</div>

# github-action-example

This repo contains an example GitHub Action that builds a Windows C++ application and uses [symbol-upload](https://github.com/BugSplat-Git/symbol-upload) to publish symbols to BugSplat. You can either use this template repo, or follow the steps below to configure an existing application.

## Steps 🥾

1. Create a new [BugSplat](https://www.bugsplat.com) database
2. Generate an OAuth2 Client ID and Client Secret pair on the [Integrations](https://app.bugsplat.com/v2/database/integrations#oauth) page
3. Create a [repository secret](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository) in your GitHub repo with the key `BUGSPLAT_DATABASE` and your BugSplat database name as the value
4. Create repository secrets for `SYMBOL_UPLOAD_CLIENT_ID` and `SYMBOL_UPLOAD_CLIENT_SECRET`
5. Configure BugSplat according to the [docs for your specific platform](https://docs.bugsplat.com/introduction/getting-started/integrations).
6. If you're using preprocessor definitions to supply a value for `BUGSPLAT_DATABASE`, be sure to configure your project file as seen [here](https://github.com/BugSplat-Git/github-action-example/blob/cd3d15eda4ed715bbe931490d32e074628dcd036/Samples/myConsoleCrasher/myConsoleCrasher.vcxproj#L169-L170).
7. Create a GitHub Action that builds your project an uploads symbols. Here's a copy of the action used by this repo

```yml
name: Build myConsoleCrasher Project

on: [push]

jobs:
  build:
    name: Build myConsoleCrasher
    runs-on: windows-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Setup MSBuild path
      uses: microsoft/setup-msbuild@v1.1

    - name: Build myConsoleCrasher project
      env:
        BUGSPLAT_DATABASE: ${{ secrets.BUGSPLAT_DATABASE }}
      run: msbuild .\Samples\myConsoleCrasher\myConsoleCrasher.vcxproj /p:configuration=release /p:DefineConstants=BUGSPLAT_DATABASE=%BUGSPLAT_DATABASE%

    - name: Symbols 📦
      uses: BugSplat-Git/symbol-upload@pwsh
      with:
        clientId: "${{ secrets.SYMBOL_UPLOAD_CLIENT_ID }}"
        clientSecret: "${{ secrets.SYMBOL_UPLOAD_CLIENT_SECRET }}"
        database: "${{ secrets.BUGSPLAT_DATABASE }}"
        application: "MyConsoleCrasher"
        version: "1.0.0"
        files: "*.{pdb,exe,dll}"
        directory: "BugSplat\\Win32\\release"
        node-version: "20"
```

8. Trigger a build and navigate to BugSplat's [Versions](https://app.bugsplat.com/v2/versions) page to verify symbols were uploaded.
9. Run your application and generate a crash report to test your BugSplat integration.

<img width="1725" alt="image" src="https://github.com/BugSplat-Git/github-action-example/assets/2646053/92c052d0-574a-4556-8cf7-1bc1b228a2f3">

## 👷 Support

If you have any additional questions, please email our [support](mailto:support@bugsplat.com) team, join us on [Discord](https://discord.gg/bugsplat), or reach out via the chat in our web application.

