---
sidebar_label: 'Integrate BlocKey APIs'
sidebar_position: 4
---

import { qrImageContent } from '/constants/docs/integrate-apis';
import QRCode from 'react-qr-code';

# BOTP Manual

We will describe step by step how setup and implement your system to integrated with BOTP APIs in order to use the BOTP services.

## Implement your APIs to setup QR

First, you have to implement an API **for generating QR image**. In particular, your users would initially scan a QR image to register the BOTP Authenticator to your service as an authenticator app. The content of the QR image must be the following URL

```md
https://your-service-website.com/api/2fa?address=YOUR_BC_ADDRESS&username=USERNAME
```

Here is the generated QR image. We recommended that the QR image has both **small size** and **low correction level** to reduce the shoulder surfing attack.

<div style={{ display: "flex", justifyContent: 'center', width: '100%', padding: '24px 0' }}>
  <QRCode size={140} value={qrImageContent} level="L" />
</div>

After a successful QR scan, BOTP system would receive this URL from the user. We would change nothing except the `address` parameter from `YOUR_BC_ADDRESS` to `USER_BC_ADDRESS`, and call that new URL. This is also the second API your system must implement, **to receive the information of user who registered BOTP Authenticator.**


```md
...?address=USER_BC_ADDRESS&username=USERNAME
```

## Integrate with BOTP APIs to validate OTP

Next, you have to integrate with our APIs to authenticate the 2FA process. But in advance, you have to **get your API-Key** in` BOTP Dashboard > Settings > Profile`

When the second factor authentication is needed, your system call the `sendMessage` API first to **send transaction message to the users**. In particular, each message contains `userAddress` (user blockchain address), `notifyMessage` (transaction message that shown up to the user), and `message` (private message to generate OTP code, and is not shown up).

```md
POST https://botp-backend-logic-api.herokuapp.com/api/v1/message/sendMessage
```

```json
{
  "APIKey": "aa8ea422-49c9-42b6-b645-b5654aa56639",
  "ObjectListParams": [
    {
      "userAddress": "0xDB026e60C1083375167094ae3531352f47f05b0F",
      "message": "keythinh1",
      "notifyMessage": "[khiem-2] Test analyser1"
    },
    {
      "userAddress": "0xC0c0b84907b5b93aAF37936eC5d9D1fDF7A60aD5",
      "message": "keythinh2",
      "notifyMessage": "[khiem-2] Test analyser2"
    }
  ]
}
```

Finally, when the user entered the OTP received from BOTP app, your system don't need to verify it by hand, but by calling our `agentValidateOTP` API. We recommended the `SHA-512` algorithm to generate OTP.

```md
POST https://botp-backend-logic-api.herokuapp.com/api/v1/otp/agentValidateOTP
```

```json
{
  "APIKey": "2c6b9e65-4018-44bf-b130-aa3e3ce7d937",
  "ObjectListParams": [
    {
      "userAddr": "0xf0465189F703fAb578e2A040C6906460463115d9",
      "OTP": "3982995",
      "message": "agennenwgwj"
    }
  ],
  "period": 120,
  "digits": 7,
  "algorithm": "SHA-512"
}
```
