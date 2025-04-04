# Yearn Protocol Documentation Website

The Yearn Docs [website](https://docs.yearn.fi/) is built using [Docusaurus](https://docusaurus.io/), a modern static website generator.

## Installation and Setup

1. Fork the yearn-devdocs repository at https://github.com/yearn/yearn-devdocs

2. clone your forked repo locally. Replace `<yourUserName>` with your github user name

    ```bash
    git clone https://github.com/<yourUserName>/yearn-devdocs
    cd yearn-devdocs
    ```

### Install project dependencies

1. Install Node dependencies

    make sure you have the most recent lts node version installed. NVM is Node Version Manager and you may need to install this if you don't have it.

    ```bash
    nvm install --lts
    ```

    ```bash
    nvm use --lts

    ```

    Then install node dependencies

    ```bash
    yarn
    ```

2. Install Python/Vyper dependencies for natspec docs generation (You can skip this step if you aren't working on smart contract documentation)

    This assumes you are using linux and will use the apt package manager. If not, other OSes have their own package manager that will have Python and Vyper.

    2a. update the apt package manager

    ```bash
    sudo apt update
    ```

    2b. Install Python 3

    ```bash
    sudo apt install python3
    ```

    2c. Verify the installation

    ```bash
    python3 --version
    ```

    2d. Create and initialize python virtual environment

    ```bash
    python3 -m venv venv
    source venv/bin/activate
    ```

    2e. install requirements

    ```bash
    python3 -m pip install -r requirements.txt
    ```

    2f. Install Foundry

    ``` bash
    curl -L https://foundry.paradigm.xyz | bash
    ```

    open a new terminal window and run

    ```bash
    foundryup
    ```

## Local Development

This command starts a local development server and opens up a browser window. Most changes are reflected live without having to restart the server.

```bash
yarn start
```

Running `yarn start` will also run a script that checks the validity of smart contract addresses in the docs against a local constants file. This runs on every dev run as a way to keep the checks updated and the information about them on the site current (all PRs will include an update to last check time). If you want to run `yarn start` without the `runAddressChecks` script running you can run `yarn start-no-check`.

## Build

This command generates static content into the `build` directory and can be served using any static content hosting service.

```bash
yarn build
```

## Configure .env

The docs site pulls data from on-chain smart contracts, so an API key is necessary. The default is an Alchemy API key so the easiest thing to do is get a free api key from them at https://www.alchemy.com/pricing.

Rename the `.env.example` file in the root directory to `.env` and add your API key where it says "yourApiKeyHere" without any quotes or backticks.

If you would like to use a different RPC service, or your own node to pull blockchain data, you can edit the publicClient in `/src/context/PublicClientContext`.

## Deployment

Generally you should not need to use this as there is a github actions script that builds and deploys the site when a pull request is merged to the `master` branch of the upstream Yearn Github Repo.

If you are using GitHub pages for hosting, this command is a convenient way to build the website and push it to the `gh-pages` branch.

```bash
GIT_USER=<Your GitHub username> USE_SSH=true yarn deploy
```

---

## Contribute

For detailed information on the contributing workflow, please see the [Contributing Documentation](CONTRIBUTING.md).

### Documentation Structure

We have 2 types of documentation: General documentation and Natspec documentation of smart contracts:

- General documentation is generated from markdown or HTML files and edited manually.
- Natspec documentation is automatically generated from another repository's code.

#### General Documentation

This documentation is located in the `docs` folder and most content is either in the `getting-started` folder (User Docs), the `developers` folder (Dev Docs), or the `contributing` folder (DAO Docs).

#### Natspec Documentation

⚠️ As of mid 2024, versioning has been changed!

Because Yearn supports multiple products with their own version numbers, versioning is now done manually using folder structure (instead of native Docusaurus versioning) to keep things organized. There are folders for all versions of the smart contract [NatSpec](https://docs.soliditylang.org/en/latest/natspec-format.html) documentation in:

```plaintext
docs/developers/smart-contracts
├── V3
│   ├── Current contract 1
│   ├── Current contract 2
│   └── Current contract n...
│
├── V2
│   ├── Current contract 1
│   ├── Current contract 2
│   └── Current contract n...
│
├── Deprecated   
│   ├── V3 Deprecated
│   │   ├── v3.x.x
│   │   │   ├── contract 1
│   │   │   ├── contract 2
│   │   │   └── contract n...
│   │   └── v3.x.x
│   │       ├── contract 1
│   │       ├── contract 2
│   │       └── contract n...
│   │
│   └── V2 Deprecated
│       ├── v0.4.x
│       │   ├── contract 1
│       │   ├── contract 2
│       │   └── contract n...
│       └── v0.4.x
│           ├── contract 1
│           ├── contract 2
│           └── contract n...
```

### Generating V2 Versioned Documentation

#### Dependencies

- Clone the V2 vaults repository: [yearn/yearn-vaults](https://github.com/yearn/yearn-vaults) in the same folder where you cloned yearn-devdocs (not inside devdocs, but beside it)
- Run the yearn-vaults [installation](https://github.com/yearn/yearn-vaults#installation), you will need to have brownie installed to run it once so it installs the required dependencies.
- Check the vyper compiler version on the vaults repo ([here](https://github.com/yearn/yearn-vaults/blob/master/contracts/Vault.vy#L1)) and update the vyper version in the `requirements.txt` file in this repo.
- Make sure [Vault.vy](https://github.com/yearn/yearn-vaults/blob/master/contracts/Vault.vy#L1) and [Registry.vy](https://github.com/yearn/yearn-vaults/blob/master/contracts/Registry.vy#L1) on `yearn-vaults` folder has the same compiler version on their first line. If not, bump the file with the lowest version to the current version the other uses.
- If any contract file in yearn-vaults uses a fixed compiler version (without leading `^`) you may have to add it so the `solc` compiler will run. Also, make sure `solc` version is up-to-date.
- If you get errors with global `solc` try to install it locally on the project with npm

#### Generate

To generate API documentation and coin a new release, do the following.

1. Create new folder in `docs/developers/smart-contracts/deprecated/V2` with the current version number (i.e. version-0.4.6)
2. Move existing docs into newly created folder. Leave the index.md file and update the vault version called out in it to the new version number.
3. Generate docs from contracts using vydoc or similar documentation creator.
4. lint and clean up docs
5. Move new generated docs into `docs/developers/smart-contracts/v2/`

#### VyDoc

Generate docs with vydoc. Arguments are:
-i is where the smart contracts live
-o is where the generated documentation will be written
-t is the template used to generate the docs
-c is the compiler (make sure you have the correct vyper installed)

```bash
npx vydoc -i ../yearn-vaults/contracts/ -o ./natspec/temp -t ./natspec/contract.ejs -c $(which vyper)
```

```bash
npx solidity-docgen@0.5.17 --solc-module solc --templates=natspec --helpers=helpers/solidityHelpers.js -i ../yearn-vaults/contracts/ -o ./docs/developers/smart-contracts/v2
```

### Generating V3 Smart Contract Documentation

1. All necessary V3 github repos should be included as git submodules within this repo. They are located in the `natspec/lib` folder. If the repo folders are empty, you need to install the submodules:

```bash
git submodule update --init --recursive
```

For reference, these are the addresses for the different submodules:

- [yearn/yearn-vaults-v3](https://github.com/yearn/yearn-vaults-v3/tree/master)
- [yearn/vault-periphery](https://github.com/yearn/vault-periphery)
- [yearn/tokenized-strategy](https://github.com/yearn/tokenized-strategy/tree/master)
- [yearn/tokenized-strategy-periphery](https://github.com/yearn/tokenized-strategy-periphery/tree/master)
- [yearn/yearn-ERC4626-Router](https://github.com/yearn/Yearn-ERC4626-Router)
  
2. If you haven't already, create a python virtual environment and initialize.

```bash
python3 -m venv venv
source venv/bin/activate
```

3. If you haven't already, install the python requirements.

```bash
python3 -m pip install -r requirements.txt
```

4. If you haven't already, install Forge.

``` bash
curl -L https://foundry.paradigm.xyz | bash
```

open a new terminal window and run

```bash
foundryup
```

then re-activate your venv

```bash
source venv/bin/activate
```

You should now be ready to generate some docs! To generate new documentation run the v3 documentation script.

```bash
yarn v3-docs
```

This should generate a prompt for you to follow:

- If you are creating docs for a new release then select "new" and enter the new version number. This will then:
  - Generate documentation for the contracts listed in the `v3-contracts` object in `smartContracts.json`.
  - Move the existing "current" files into a new folder in `developers/smart-contracts/deprecated/V3` with the old version number.
  - Add the new files to `developers/smart-contracts/v3`
  - update the index.md file to reflect the new release version.
- If you only want to update without creating a new version, or update a few files or a certain folder, you can use the "update" option in the prompt.
  - Then select whether you want to update all v3 files or only selected ones.
    - If you select "files", the script will then read from the `selectedFiles` array in the `natspec-generate.mjs` script file. Edit that array to include the file names you want. (Naming should match existing files e.g. `TokenizedStrategy`).
    - If you select "folder" then the script will read the path in the `folderToUpdate` variable. Be aware this will include all sub-folders as well.
  - After selecting options, everything else runs the same as selecting "new" but without copying your existing files to the deprecated folder.

The script runs a markdown linter as well as some regex to remove elements that break MDX and some broken links, so the output files should be pretty clean, but they still may need some manual adjustment. You may also still get build errors if there are characters in the natspec that MDX v3 doesn't like (like {} and <>). These will need to be removed manually or escaped out of using the `\` character. More info [here](https://docusaurus.io/docs/markdown-features/react#markdown-and-jsx-interoperability). Steps to fix these types of errors:

There will also probably be some broken links. These are usually from the forge doc build and will reference the structure of the forge doc output. Easiest way to find these is to do a search in VSCode. I search for `(/src/` with `developers/smart-contracts/v3/*` included and `developers/smart-contracts/v3/deprecated/*` excluded. The most common broken links will be links generated in Forge that link to other sections in the same file. These use the hash router and usually just need everything before the hash to be removed to work. i.e. `[redeem](/src/Yearn4626Router.sol/contract.Yearn4626Router.md#migrate)` -> `[redeem](#migrate)`.

---

### Custom Elements

#### Detail Element

This is a Detail element that contains other text inside. If you format the summary section as shown it renders markdown correctly.

```markdown
<details className="customDetails">

  <summary>
  
  ## Title Here
  
  </summary>

### Subtitles as needed

content here

</details>
```

There is also a "customFaqDetails" css class that removes the borders.

#### PrettyLink

The PrettyLink element makes your links into button-like elements with subtle animation and yearn styling. These links will fill the full width of the markdown document. Can be used with naked links or with markdown style links.

```markdown
<PrettyLink>[your link name](your-link-url)</PrettyLink>
```

#### Yearn Admonition

There are custom informational Yearn-styled admonitions that can be used like any other admonition.

```markdown
:::yearn[title-goes-here]

text content

:::
```

```markdown
:::yearn-data[title-goes-here]

text content

:::
```

If you create a new docs plugin in `docusaurus.config` you will need to initialize this admonition in that plugin instance like this:

```js

 plugins: [
    [
      '@docusaurus/plugin-content-docs',
      {
        id: 'new docs section',
        //other vars
        admonitions: {
          keywords: ['yearn', 'yearn-data'],
          extendDefaults: true,
        },
      },
    ],
]

```

### Blockchain RPC Calls

You can make RPC calls to read contract data from on-chain sources and display them within the docs. This is done using the Viem ethereum library. But if all you are doing is writing docs, you don't need to worry about the details here. You can add the information for all the read calls you want within the front-matter of a markdown document. [Front-matter](https://docusaurus.io/docs/markdown-features#front-matter) is metadata that docusaurus reads when serving pages.

To make a blockchain call you need to structure your data in the following format:

```yml
---
rpcCalls:

  - name: 'dYFI Redemption' <-- descriptive name of contract to be called for use in component
    chain: '1' <--chainID
    address: '0x7dC3A74F0684fc026f9163C6D5c3C99fda2cf60a' <--the contract address
    abiName: 'dyfiRedemptionABI' <--name of exported ABI object from src/ethereum/ABIs
    methods:  
      - 'discount' <-- name of call (if no arguments needed)
      - 'get_latest_price'
      - name: 'eth_required' <-- name of call (if arguments are needed)
        args: ['1000000000000000000'] <--comma separated arguments of call as an array (square brackets)

  - name: 'YFI token'
    chain: '1'
    address: '0x0bc529c00C6401aEF6D220BE8C6Ea1667F6Ad93e'
    abiName: 'yfiTokenABI'
    methods:
      - totalSupply
      - symbol
---
```

- Each element in rpcCalls creates an object that is exported from the front-matter.
- Each object can contain calls to different functions in the same contract if you add them into the methods field.
  - If the method doesn't require args (the function doesn't require any input) then you only need to list the name. This is the name you see on etherscan, without the number.
  - If the method does require an argument then you need to add it with a name parameter and an args parameter, with the values for the arguments separated by commas and in square brackets.

![name on etherscan](static/img/guides/readme/etherscan1.png)

![name and args on etherscan](static/img/guides/readme/etherscan2.png)

⚠️ When adding a new contract to call, you need to add the ABI to "src/ethereum/ABIs". Create a new typescript file with the name of your ABI. The convention is to name it in camelCase and end it with ABI (i.e. yfiTokenABI.ts). Then paste the ABI into the file (you can copy it from etherscan. It is in the contract->code section.). You need to export it and add `as const` at the end.

```js
// src/ethereum/ABIs/yourContractABI.ts
export const yourContractABI = [
    {
    // abi data here
    }
] as const

```

You then need to export this element from the index.ts file in the same directory. Add a line exporting your ABI as shown below.

```js
export * from './yourContractABI'
```

To display the data from the calls, use the \<ContractData> component. It takes the following arguments:

- `contractName` which reads the name field in the rpcCall defined in the front-matter
- `methodName` which reads from the methods in the rpcCall defined in the front-matter
- `decimals` which is an optional argument to format your output to display with human readable decimals. It should be wrapped in curly brackets \{\}.

```markdown
The current redemption discount is: <ContractData contractName='dYFI Redemption' methodName='discount' decimals={18} />
```
