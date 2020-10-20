# Capistrano actions
Github deploy action for Capistrano. Use this action to automate your capistrano deployment process.

## Dependencies
Ruby should be installed with the official ruby action (https://github.com/ruby/setup-ruby)

## Inputs
### `target`
Environment where deploy is to be performed to. E.g. "production", "staging". Default value is empty

### `deploy_key`
**Required** Symmetric key to decrypt private RSA key. Must be a string.

### `enc_rsa_key_pth`
Path to the encrypted key. Default `"config/deploy_id_rsa_enc"`. You have to use either `enc_rsa_key_pth` or `enc_rsa_key_val`.

### `enc_rsa_key_val`
Contents of the encrypted key. Best to use as repository secret. You have to use either `enc_rsa_key_pth` or `enc_rsa_key_val`.

### `working-directory`
The directory from which to run the deploy commands, including `bundle install`.

### `bundler_version`

The version of Bundler to install before running the bundle-command. By default Bundler v2.1.4 is used.

## Outputs
No outputs

## Setting up CD using this action
1. Generate SSH keys on the target machine
```bash
$ ssh-keygen
```
2. Export public key to the `authorized_keys` to allow the usage of this keypair to login
```bash
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
3. Add public key from `~/.ssh/id_rsa.pub` to your repository's deployment keys via *Settings / Deploy keys / Add*
4. Encrypt your private key with a strong password. **Please use these options**, otherwise this action may not be able to decrypt your key.
```bash
$ openssl enc -aes-256-cbc -md sha512 -salt -in .ssh/id_rsa -out deploy_id_rsa_enc -k PASSWORD -a
```
5. Add `deploy_id_rsa_enc` file to your repository. Suggested path is `config/deploy_id_rsa_enc`
6. Save the password used in step 4 as a secret in repository settings via *Settings / Secrets / Add*
7. Create YAML configuration for your workflow (example below)

## Workflow example
```yaml
name: Deploy with Capistrano

on:
  push:
    branches:
    - master

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout repository
      uses: actions/checkout@v2
    - name: Set up Ruby
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: 2.6 # Not needed with a .ruby-version file
    - name: Restore Bundler cache
      id: cache
      uses: actions/cache@v1
      with:
        path: vendor/bundle
        key: ${{ runner.os }}-bundle-${{ hashFiles('**/Gemfile.lock') }}
        restore-keys: |
          ${{ runner.os }}-bundle-
    - name: Deploy application
      uses: IQTHINK/capistrano-deploy@v2.2
      with:
        target: production
        deploy_key: ${{ secrets.DEPLOY_ENC_KEY }}
        bundler_version: 2.1.0 # default 2.1.4
```
