# Using Plan/Action Crytography

## Overview

Plans support encrypting ParameterInfo blocks (Config, Parameters). You will ned to supply a set of one or more RSA keys, encrypt the plan using synapse.cli (pre-execution), and locate the keys at the executing Nodes for runtime decryption support.

1. Use `synapse.cli -> genkey` to [generate](#creating-rsa-keys) an RSA key pair.
2. Declare [Plan](#plan-settings) and [Action](#action-settings) Crypto settings, naming the RSA keys from step 1 (or other keys, as desired)
3. [Encrypt](#encryptingdecrypting-a-plan) the Plan
4. Configure [Synapse.Node](#integrating-an-encrypted-plan-with-synapseserver) with RSA keys from step 1

## Creating RSA Keys

To create an RSA public/private key pair, use the `synapse.cli -> genkey` option.

```
synapse.cli.exe genkey:{filePath} [kcn:{keyContainerName}]

 - Create RSA keypair for use in encrypt/decrypt actions.

 genkey       - filePath: Valid path to create keys.
 kcn          - keyContainerName: Optional container within file.
              If keyContainerName is specified, it must be used in
                encrypt/decrypt actions (specified in Plans settings).
```

This will generate two files: [path]\filename.ext.**pubOnly** (public key only) and [path]\filename.ext.**pubPriv** (public and private key, both).  The keyContainer is an internal mechanism used in key storage.  You will specify the same keyContainer when encrypting/decrypting plans.

## Declaring Crypto Settings

Both Plans and Plan Actions support declaring Crypto blocks, where Plan settings are inherited by Actions, and any Action can override the inheritance by declaring local settings.  Local Action settings _are not_ downwardly inherited by child Actions.  Importantly, Plan-level declaration does not support encrypting individual ParameterInfo block elements, that is left to the Action Crypto settings.

### Plan Settings

At the Plan level, you may declare the KeyUri and KeyContainerName.  These settings will, by default, be inherited by all Actions within the Plan, unless the Action overrides them.  Any Plan-level Crypto->Element settings will be ignored.

```yaml
Name: crypto_sample
Description: Test Plan Encryption
Crypto:
  Key:
    Uri: #file or http path to the pubPriv file.
    ContainerName: #name specified when creating the keys.
Actions:
  ...
```

### RunAs Settings

At the Plan and Action level, you may declare local Crypto settings within a RunAs block.  By default, RunAs->Crypto blocks will inherit the Plan->Crypto settings.

```yaml
RunAs:
  Domain: myDomain
  UserName: myUser
  Password: qbMVEA84crEncryptedStringQ2eFbJ/fvs=
  Crypto:
    Key:
      Uri: 'C:\crypto\pubPriv.xml'
      ContainerName: myContainer
    Elements:
    - Password
  IsInheritable: true
  BlockInheritance: true
```


### Action Settings

At the Action level, you will declare a list of Elements within a ParameterInfo block (Config, Parameters) to encrypt/decrypt.  It is technically possible to encrypt/decrypt any part of a ParameterInfo block, but a typical implmentation only requires encrypting/decrypting the Values section, as it might contain staticly declared sensitive values.

```yaml
Name: crypto_sample
Description: Test Plan Encryption
Crypto:
  Key:
    Uri: #file or http path to the pubPriv file.
    ContainerName: #name specified when creating the keys.
Actions:
- Name: action_name
  ...
  Handler:
    ...
    Config:
      ...
      Crypto:
        Key:
          Uri: #{optional setting, overides Plan-level declaration}
          ContainerName: #{optional setting, overides Plan-level declaration}
        Elements:
        - Type
        - Values:a:b
        - Values:a:c[0]:d
        - Values:a:c[1]:d:e:f[1]:g
      Values:
        ...
  Parameters:
    Crypto:
      Key:
        Uri: #{optional setting, overides Plan-level declaration}
        ContainerName: #{optional setting, overides Plan-level declaration}
      Elements:
      - Type
      - Values:a:b
      - Values:a:c[0]:d
      - Values:a:c[1]:d:e:f[1]:g
    Values:
      ...
```

## Encrypting/Decrypting a Plan

Once you've created RSA keys and declared Crypto settings within a Plan, use `synapse.cli` to encrypt/decrypt the Plan itself.

```dos
synapse.cli.exe encrypt|decrypt:{filePath} [out:{filePath}]

 - Encrypt/decrypt Plan elements based on Plan/Action Crypto sections.

 encrypt      - filePath: Valid path to plan file to encrypt.
 decrypt      - filePath: Valid path to plan file to decrypt.
 out          - filePath: Optional output filePath.
              If [out] not specified, will encrypt/decrypt in-place.
```

## Integrating an Encrypted Plan with Synapse.Server

Decrypting Plans at runtime is executed at Synapse.Node.  As such, Synapse.Node needs access to the public/private keypair used to encrypt the plan.  This may be accomplished by:

1. Placing a copy of the keys local to the Node, typically in the Cryto folder, or
2. Placing a copy of the keys in a secure, central location, such as shared drive/bucket/storage (under file:// or http:// location).

    - Be sure the runtime security context has access to the shared location.  Recall that security may be configured at the Node itself, declared as a Plan/Action RunAs context, or passed from the Controller for straight-through impersonation.  Check the Plan.ResultPlan to see an audit history of the execution security context. See [Authentication](./setup/authentication), [Impersonation](./setup/impersonation), and [RunAs Security Context](./../plans/actions/detail/#runas) for detail.

At runtime, Synapse.Node will execute just-in-time decryption, hands-off decrypted values to a handler, then immediately disposes of in-memory decrypted values.