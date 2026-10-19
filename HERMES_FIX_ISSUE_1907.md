# Solution for Issue #1907

## Summary
In `decodeCctpMessage`, strict version validation checks have been implemented for `version` (CCTP message header) and `burnBodyVersion` (burn message body payload) against the supported protocol version (`0`). When an unsupported header or payload version is received, the decoder immediately throws an `Unsupported message version` error. This prevents corrupted parsing of payload fields caused by shifted schema offsets and avoids the execution of invalid cross-chain transactions.

## Implementation / Patch
```diff
--- a/src/utils/message.ts
+++ b/src/utils/message.ts
@@ -10,6 +10,10 @@
+export const SUPPORTED_CCTP_MESSAGE_VERSION = 0;
+export const SUPPORTED_BURN_BODY_VERSION = 0;
+
 export const MESSAGE_VERSION_OFFSET = 0;
 export const MESSAGE_VERSION_SIZE = 4;
@@ -88,14 +92,26 @@ export function decodeCctpMessage(rawMessage: Uint8Array | Hex): DecodedCctpMess
   if (messageBytes.length < MIN_CCTP_MESSAGE_LENGTH) {
     throw new Error(`Invalid message length: ${messageBytes.length}. Minimum required: ${MIN_CCTP_MESSAGE_LENGTH}`);
   }
 
+  // Read and validate header message version
+  const version = readUint32BE(messageBytes, MESSAGE_VERSION_OFFSET);
+  if (version !== SUPPORTED_CCTP_MESSAGE_VERSION) {
+    throw new Error(
+      `Unsupported CCTP message version: ${version}. Supported version is ${SUPPORTED_CCTP_MESSAGE_VERSION}.`
+    );
+  }
+
   const sourceDomain = readUint32BE(messageBytes, SOURCE_DOMAIN_OFFSET);
   const destinationDomain = readUint32BE(messageBytes, DESTINATION_DOMAIN_OFFSET);
   const nonce = readUint64BE(messageBytes, NONCE_OFFSET);
   const sender = readBytes32(messageBytes, SENDER_OFFSET);
   const recipient = readBytes32(messageBytes, RECIPIENT_OFFSET);
   const destinationCaller = readBytes32(messageBytes, DESTINATION_CALLER_OFFSET);
 
   const messageBody = messageBytes.slice(MESSAGE_BODY_OFFSET);
 
+  // Read and validate burn body version
+  const burnBodyVersion = readUint32BE(messageBody, BURN_BODY_VERSION_OFFSET);
+  if (burnBodyVersion !== SUPPORTED_BURN_BODY_VERSION) {
+    throw new Error(
+      `Unsupported CCTP burn body version: ${burnBodyVersion}. Supported version is ${SUPPORTED_BURN_BODY_VERSION}.`
+    );
+  }
+
   const burnToken = readBytes32(messageBody, BURN_TOKEN_OFFSET);
   const mintRecipient = readBytes32(messageBody, MINT_RECIPIENT_OFFSET);
   const amount = readUint256BE(messageBody, AMOUNT_OFFSET);
   const messageSender = readBytes32(messageBody, MESSAGE_SENDER_OFFSET);
 
   return {
     version,
     sourceDomain,
     destinationDomain,
     nonce,
     sender,
     recipient,
     destinationCaller,
     burnBody: {
       version: burnBodyVersion,
       burnToken,
       mintRecipient,
       amount,
       messageSender,
     },
   };
 }
```

## Payout Address (Solana)
`3EZnNP2kfmY161XyqnRWEFgtYw6jWFXHnimU5RMqA8g8`
