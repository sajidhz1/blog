---
layout: post
title: "Bouncy Castle library How to Specify Additional Parameters to the Private key"
modified:
categories: blog
excerpt:
tags: [Bouncy Castle, Windows, Certificate, X509Certificate ,C#]
image:
  feature:
date: 2017-08-12T08:08:50-04:00
---

[BouncyCastle](http://www.bouncycastle.org/csharp/) is a library Which will Help you to Create X509Certificates on the fly using C#. In this Example we will be focusing on how to add `KeyStoreLocation` and the `KeyContainerName` as the Additional Parameters to the Private Key.

###### 1.How to Create Certificate using Bouncy Castle 

[StackOverflow](https://stackoverflow.com/a/22247129/1577428) 

[Roger's Blog](blog.differentpla.net/blog/2013/03/18/using-bouncy-castle-from-net)


###### 2.Add additional paramters to the PrivateKey

The main usage of setting up additional parameters to the `Privatekey` is we can directly pick the private key from the store by using these parameters, so that it could be used to decrypting Values which are signed by the public key

Last and usual Step of creating a bouncy castle certificate would be to converting bouncy castle PrivateKey to system Certification PrivateKey, and we do that by converting bouncy castle private key params ro RSAparams and Assigning it to the Private key

```
x509.PrivateKey = DotNetUtilities.ToRSA(rsaparams);
```

But in this case we need to create a new Private Key using system Certificate methods and add the keyContainerName and the CsproviderFlag by creating a new CspParameter object, also instead of using `DotNetUtilities` we will be importing the Bouncy castle private key params to the `System.Security` Private key

```
//convert bouncy castle paramters to system certification parameters
 RSAParameters rsaParameters = DotNetUtilities.ToRSAParameters(rsaparams);

//create new parameters which we will use to identify the private key
       CspParameters cspParameters = new CspParameters();
            cspParameters.Flags = CspProviderFlags.UseMachineKeyStore;
            cspParameters.KeyContainerName = containerName;

//initialize a new private key using System Certifcation methods and the parameters
            RSACryptoServiceProvider rsaKey = new RSACryptoServiceProvider(keyStrength, cspParameters);
            rsaKey.PersistKeyInCsp = true;

//import Bouncy castle private key parameters to the new private key 
            rsaKey.ImportParameters(rsaParameters);

//assign it to the windows Cert 
            x509.PrivateKey = rsaKey;

```

###### 3.Usage

By parsing the Key Container value and encrypted Value we get a byte array of values decrypted by the private key 

```
public static byte[] DecryptValues(string keyContainer, byte[] encryptedValue)
{
    using (RSACryptoServiceProvider RSAPrivate = new RSACryptoServiceProvider(new CspParameters()
        {
            Flags = CspProviderFlags.UseMachineKeyStore,
            KeyContainerName = keyContainer
        }))
    {
        return RSAPrivate.Decrypt(encryptedValue, true);
    }
}

```

## Hope this Helped you! Thank You :)