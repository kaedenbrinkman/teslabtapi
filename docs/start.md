---
sidebar_position: 2
---

# Getting Started
:::tip
I really recommend reading over this to grasp an understanding of how this stuff works, but if you just want to see what the car can do go to [more info](#more-info)
:::

## Whitelisting your key
To begin working with the vehicle's BLE API, you'll need to generate and whitelist your public key with the vehicle.

To do this, you'll need to first of all, generate an EC private key with the NISTP256 curve (aka secp256r1, or prime256v1), which you should store and keep safe, this key is used to sign all your messages. From that, you'll need to generate a public key serialized to bytes in the ANSI X9.62 (also sometimes called X9.63) Uncompressed Point format (if done correctly, the first byte should always be 0x04. Also, I never tried it, but Compressed Point might work too, that is where the first byte is 0x02 or 0x03, please tell me if it does so that I can update the documentation! If you're feeling wild and try out any other public key encodings and they work, then please notify me too!), for now I'll call these `privateKey`, and `publicKey` respectively. Next serialize an unsigned protobuf message from the VCSEC protobuf in the following layout:
```
UnsignedMessage {
	WhitelistOperation {
		addKeyToWhitelistAndAddPermissions {
			key {
				PublicKeyRaw: <publicKey>
			}
			permission: WHITELISTKEYPERMISSION_LOCAL_DRIVE
			permission: WHITELISTKEYPERMISSION_LOCAL_UNLOCK
			permission: WHITELISTKEYPERMISSION_REMOTE_DRIVE
			permission: WHITELISTKEYPERMISSION_REMOTE_UNLOCK
		}
		metadataForKey {
			keyFormFactor: KEY_FORM_FACTOR_ANDROID_DEVICE
		}
	}
}
```
You may add any other permissions as needed but these are the ones that let you unlock, and start the vehicle. I'll call the previously serialized message the `protoMsg`. Once you have it serialized to bytes, you need to make a ToVCSECMessage message of the following format:
```
ToVCSECMessage {
	signedMessage {
		protobufMessageAsBytes: <protoMsg>
		signatureType: SIGNATURE_TYPE_PRESENT_KEY
	}
}
```
Serialize this message to bytes, and I'll call it `authMsg`.
From now on I'll be referencing a function which I'll call `prependLength`. What it does, is take length of some byte array which in most/all cases will be the final serialized message, extend it by 2 bytes, shift all the bytes to the right twice so that the first 2 bytes are empty. Now set the first 2 bytes to the length of the byte array passed into the function (in big-endian, so the least significant byte comes last). So what you get is this:
```python
someMsg = b'\x01\x02'
prependedMsg = prependLength(someMsg)
print(prependedMsg) # b'\x00\x02\x01\x02'
```
Once you have serialized `protoMsg`, and prepended the length, send the message to the vehicle over a normal BluetoothLE connection on the following write characteristic:
```
Serivce: 00000211-b2d1-43f0-9b88-960cebf8b91e
UUID: 00000212-b2d1-43f0-9b88-960cebf8b91e
Descriptor: 0x2901
```
The vehicle's BLE name should match this regular expression written by me:
```
/ ^S[a-f\d]{16}[A-F]$ /
```
It means that the first letter has to be "S", the next 16 characters are all either "a", "b", "c", "d", "e", "f", "0", "1", "2", "3", "4", "5", "6", "7", "8", or "9", and the last letter is either "A", "B", "C", "D", "E", or "F".

Whenever the vehicle responds, it should always respond with a FromVCSECMessage message, which after removing the first 2 bytes (which are the length of the message), you can decode. In this case the vehicle should respond with the following message:
```
FromVCSECMessage {
	commandStatus {
		operationStatus: OPERATIONSTATUS_WAIT
		signedMessageStatus {
		}
	}
}
```
It should be received on the following indication charachteristic:
```
Serivce: 00000211-b2d1-43f0-9b88-960cebf8b91e
UUID: 00000213-b2d1-43f0-9b88-960cebf8b91e
Descriptor: 0x2901
```
Now tap an existing key card, and you should recieve the following message:
```
FromVCSECMessage {
	commandStatus {
		signedMessageStatus {
		}
	}
}
```
Congrats! You whitelisted your first key!

## Getting the ephemeral key
To sign messages to send to the vehicle, you'll need to get the vehicle's ephemeral key, which according to the name should change every so often, but I never experienced it change in the few days of testing that I've done.

First of all you'll need to generate the key id, which I'll be calling `keyId`. You can generate that by doing a SHA1 digest of `publicKey`, and taking the first 4 bytes of the digest.

Now that you have your key id, you'll need to request the vehicle's ephemeral key. You can do this by making a ToVCSECMessage message with the following layout:
```
ToVCSECMessage {
	unsignedMessage {
		InformationRequest {
			informationRequestType: INFORMATION_REQUEST_TYPE_GET_EPHEMERAL_PUBLIC_KEY
			keyId {
				publicKeySHA1: <keyId>
			}
		}
	}
}
```
You can serialize this message to bytes and put it into a variable which I'll call `getEphemeralBytes`. Now do `prependLength(getEphemeralBytes)`, and send the value returned to the vehicle. The vehicle should respond with a message which you need to decode. It should looks something like this:
```
FromVCSECMessage {
	sessionInfo {
		publicKey: <vehicle's ephemeral key>
	}
}
```
When you recieve it, it should be in the ANSI X9.62 (also sometimes called X9.63) Uncompressed Point format, with the NISTP256 format, the same format that your public key was sent in. Once you load it into a variable, I'll call it `ephemeral_key`. What you now need to do is generate an AES secret from the vehicle's ephemeral public key, and your secret key. Now you need to make a SHA1 of it, and put the first 16 bytes in a variable which I'll call `sharedKey`.

## Authenticating
For the vehicle to know that you are connected, and to be able to send + receive messages, you need to authenticate yourself. To do so, you'll need to generate an authentication message in the following format:
```
UnsignedMessage {
	authenticationResponse {
		authenticationLevel: AUTHENTICATION_LEVEL_NONE
	}
}
```
Set a variable called `counter` to 1 (can be any number that hasn't been used as the counter for this key and must be >= 1), which you should increment each time after using. Now, you need to encrypt this message using the `sharedKey` with AES encryption in GCM mode, with a nonce of the `counter`, split into 4 bytes in big-endian, where the least significant byte is at the end, so say counter is 23, the nonce will be `b'\x00\x00\x00\x17'`. You should also separate the encrypted/signed message into 2 variables, `encryptedMsg` (from bytes 0 to `length` - 16), and `msgSignature` (from bytes `length` - 16 to `length`). Now you need to generate a message to send to the vehicle:
```
ToVCSECMessage {
	signedMessage {
		protobufMessageAsBytes: <encryptedMsg>
		signature: <msgSignature>
		counter: <counter>
		keyId: <keyId>
	}
}
```
Now serialize this message to bytes and pass it to the `prependLength` function, and send that to the vehicle. Now as long as you stay connected to the vehicle's BLE, your key will stay marked in the vehicle as an active key.

## Authenticating for other things
If you want to let the vehicle do things automatically without sending messages to do things (i.e. like the Tesla App does when you're next to/inside the vehicle), you can send it an authentication response of a higher level to give permission to do everything under and including that level, so say `level = 'UNLOCK'`, you are only letting the vehicle unlock, but say you do `level = 'DRIVE'`, you can unlock *or* drive. Also, everytime the vehicle does something automatically, the vehicle resets level to `NONE`.

All that you have to do is serialize and sign an auth message where the distance is optional, but if not sent to the vehicle, it will just automatically assume that you're next to/inside the vehicle:
```
UnsignedMessage {
	AuthenticationResponse {
		authenticationLevel: AUTHENTICATION_LEVEL_<LEVEL>
		estimatedDistance: <distance> # optional
	}
}
```
Once you sign that message, I'll call it `signedAuthMsg`, and its signature `signedAuthSign`, and turn it into a ToVCSECMessage message like this, which you then serialize, prepend the length, and send to the vehicle:
```
ToVCSECMessage {
	signedMessage {
		protobufMessageAsBytes: <signedAuthMsg>
		signature: <signedAuthSign>
		counter: <counter>
		keyId: <keyId>
	}
}
```

:::note
I recommend only changing this to other auth levels when the car requests it. Here's an example of a message you may get from the car, requesting to unlock it:
```
FromVCSECMessage {
	authenticationRequest {
	sessionInfo {
		token: "some random token that i don't know the use of"
	}
	requestedLevel: AUTHENTICATION_LEVEL_DRIVE
	}
}
```
In this case you'll just send the car back the following message:
```
UnsignedMessage {
	AuthenticationResponse {
		authenticationLevel: AUTHENTICATION_LEVEL_DRIVE
	}
}
```
Don't worry about the walk-away car lock, when you walk away the car will automatically lock
:::
## Sending manual actions
Say a user interacts with an app or needs to do something that can't be done automatically. In that case you need to send an RKE action. You can send those by making a message in the following format, signing it, prepending length, and sending it to the vehicle like with any other signed message:
```
UnsignedMessage {
	RKEAction_E: <any rke action>
}
```

## More Info
For more info, you can begin looking at [ToVCSECMessage](tovcsec) ([UnsignedMessage](other/unsignedmsg) in particular) to see things you can send to the car, and [FromVCSECMessage](fromvcsec) to see things you receive to the car.

Don't forget, VCSEC, is the vehicle's secondary security system, so you send **To**VCSECMessage messages, and the vehicle sends you back **From**VCSECMessage messages.

:::note
There is only one exception to this rule, that is when you added your key as keyfob, in which case you would use FromKeyfobMessage instead of ToVCSECMessage, and ToKeyfobMessage instead of FromVCSECMessage.

There is also an exception for tire pressure sensors, but I have absolutely no idea of how they work, and don't have time to research.
:::
