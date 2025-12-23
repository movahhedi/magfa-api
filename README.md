# Magfa SMS API Client

A modern, fully-typed JS/TS/ESM/Node.js client for the Magfa SMS API v2.

[![npm version](https://badge.fury.io/js/magfa-api.svg)](https://www.npmjs.com/package/magfa-api)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Features

- 🚀 **Modern**: Built with TypeScript and modern JavaScript
- 📦 **Multiple Formats**: CommonJS, ES Modules, and UMD builds
- 🔒 **Fully Typed**: Complete TypeScript definitions included
- ⚡ **Promise-based**: Async/await support out of the box
- 🛡️ **Error Handling**: Comprehensive error messages in Persian
- 🔄 **Multiple Encodings**: Support for Persian, English, 8-bit, and Binary

## Installation

```bash
npm install magfa-api
```

or

```bash
yarn add magfa-api
```

## Quick Start

```typescript
import Magfa from 'magfa-api';

const magfa = new Magfa(
  'username',    // Your Magfa username
  'password',    // Your Magfa password
  'domain',      // Your domain
  '3000xxxxx'    // Your sender number
);

// Send a simple SMS
const result = await magfa.Send({
  recipients: '09123456789',
  messages: 'سلام، این یک پیام تستی است'
});

console.log(result);
```

## API Reference

### Constructor

```typescript
new Magfa(username: string, password: string, domain: string, from: string)
```

Creates a new Magfa client instance.

**Parameters:**
- `username` (string): Your Magfa account username
- `password` (string): Your Magfa account password
- `domain` (string): Your Magfa domain
- `from` (string): Default sender number for outgoing messages

**Example:**
```typescript
const magfa = new Magfa('myuser', 'mypass', 'mydomain', '3000xxxxx');
```

### Methods

#### Send

Send one or multiple SMS messages.

```typescript
Send(options: {
  recipients: string | string[];
  messages: string | string[];
  encodings?: Encoding | Encoding[];
  uids?: number[];
  udhs?: string[];
}): Promise<ISendResponseWithCount>
```

**Parameters:**
- `recipients` (string | string[]): Phone number(s) to send to (e.g., '09123456789' or ['09123456789', '09187654321'])
- `messages` (string | string[]): Message text(s) to send
- `encodings` (optional): Encoding type(s) for messages (default: Auto-detect)
- `uids` (optional): Unique identifiers for tracking messages
- `udhs` (optional): User Data Header for advanced messaging

**Returns:**
```typescript
{
  status: number;
  messages: ISentMessage[];
  count: {
    success: number;
    fail: number;
  };
}
```

**Examples:**

Send a single message:
```typescript
const result = await magfa.Send({
  recipients: '09123456789',
  messages: 'Hello World'
});
```

Send multiple messages to multiple recipients:
```typescript
const result = await magfa.Send({
  recipients: ['09123456789', '09187654321'],
  messages: ['Message for first recipient', 'Message for second recipient']
});
```

Send the same message to multiple recipients:
```typescript
const result = await magfa.Send({
  recipients: ['09123456789', '09187654321', '09351234567'],
  messages: 'This message goes to everyone'
});
```

Send with specific encoding:
```typescript
import { Encoding } from 'magfa-api';

const result = await magfa.Send({
  recipients: '09123456789',
  messages: 'سلام دنیا',
  encodings: Encoding.Persian
});
```

Send with unique IDs for tracking:
```typescript
const result = await magfa.Send({
  recipients: ['09123456789', '09187654321'],
  messages: ['Message 1', 'Message 2'],
  uids: [1001, 1002]
});
```

#### Statuses

Get delivery status of sent messages.

```typescript
Statuses(mids: number[]): Promise<IMagfaResponse_Statuses>
```

**Parameters:**
- `mids` (number[]): Array of message IDs to check status for

**Returns:**
```typescript
{
  status: number;
  dlrs: Array<{
    mid: number;        // Message ID
    status: number;     // Delivery status code
    date: string;       // Date in 'yyyy-mm-dd hh:mm:ss' format
  }>;
}
```

**Example:**
```typescript
const statuses = await magfa.Statuses([123456, 123457, 123458]);

statuses.dlrs.forEach(dlr => {
  console.log(`Message ${dlr.mid}: Status ${dlr.status} at ${dlr.date}`);
});
```

**Status Codes:**
- `-1`: ID not found (invalid ID or more than 24 hours passed)
- `0`: No status received yet
- `1`: Delivered to phone
- `2`: Not delivered to phone
- `8`: Delivered to operator
- `16`: Not delivered to operator

#### Mid

Get message ID by unique identifier.

```typescript
Mid(uid: number): Promise<IMagfaResponse_Mid>
```

**Parameters:**
- `uid` (number): The unique identifier you provided when sending the message

**Returns:**
```typescript
{
  status: number;
  mid: number | undefined;  // Message ID
}
```

**Example:**
```typescript
// First send with UID
await magfa.Send({
  recipients: '09123456789',
  messages: 'Test message',
  uids: [1001]
});

// Later retrieve the message ID
const result = await magfa.Mid(1001);
console.log(`Message ID: ${result.mid}`);
```

#### Balance

Check your account balance.

```typescript
Balance(): Promise<IMagfaResponse_Balance>
```

**Returns:**
```typescript
{
  status: number;
  balance: number | null;  // Balance in Rials
}
```

**Example:**
```typescript
const result = await magfa.Balance();
console.log(`Current balance: ${result.balance} Rials`);
```

#### ReceivedMessages

Retrieve received messages.

```typescript
ReceivedMessages(count?: number): Promise<IMagfaResponse_ReceivedMessages>
```

**Parameters:**
- `count` (number, optional): Number of messages to retrieve (default: 100)

**Returns:**
```typescript
{
  status: number;
  messages: Array<{
    body: string;              // Message content
    senderNumber: string;      // Sender phone number
    recipientNumber: string;   // Recipient phone number (your number)
    date: string;              // Date in 'yyyy-mm-dd hh:mm:ss' format
  }>;
}
```

**Example:**
```typescript
// Get last 50 received messages
const result = await magfa.ReceivedMessages(50);

result.messages.forEach(msg => {
  console.log(`From ${msg.senderNumber}: ${msg.body}`);
});
```

## Types and Interfaces

### Encoding

Enum for message encoding types:

```typescript
enum Encoding {
  Auto = 0,      // Auto-detect (default)
  Persian = 2,   // Persian/Farsi
  EightBit = 5,  // 8-bit
  Binary = 6     // Binary
}
```

### ISentMessage

Information about a sent message:

```typescript
interface ISentMessage {
  status: number;        // Status code (0 = success)
  id: number;            // Unique message ID
  userId: number;        // User ID
  parts: number;         // Number of message parts
  tariff: number;        // Message cost
  alphabet: "DEFAULT" | "UCS2";  // Encoding type
  recipient: string;     // Recipient phone number
  message?: string;      // Status message (in Persian)
}
```

### ISendResponseWithCount

Response from Send method:

```typescript
interface ISendResponseWithCount {
  status: number;
  messages: ISentMessage[];
  count: {
    success: number;     // Number of successfully sent messages
    fail: number;        // Number of failed messages
  };
}
```

## Error Handling

The package includes a custom `MagfaError` class that provides detailed error messages in Persian:

```typescript
import Magfa, { MagfaError } from 'magfa-api';

try {
  const result = await magfa.Send({
    recipients: '09123456789',
    messages: 'Test message'
  });
} catch (error) {
  if (error instanceof MagfaError) {
    console.error(`Magfa Error [${error.status}]: ${error.message}`);
  } else {
    console.error('Unknown error:', error);
  }
}
```

### Common Error Codes

| Code | Description (Persian) |
|------|----------------------|
| 1 | شماره گیرنده نادرست است |
| 2 | شماره فرستنده نادرست است |
| 14 | مانده اعتبار ریالی مورد نیاز برای ارسال پیامک کافی نیست |
| 16 | حساب غیرفعال است |
| 17 | حساب منقضی شده است |
| 18 | نام کاربری و یا کلمه عبور نامعتبر است |
| 20 | شماره فرستنده به حساب تعلق ندارد |

[See full error codes list](#error-codes-reference)

## Advanced Usage

### Batch Sending with Progress Tracking

```typescript
async function sendBatch(recipients: string[], message: string) {
  const batchSize = 100;
  const results = [];

  for (let i = 0; i < recipients.length; i += batchSize) {
    const batch = recipients.slice(i, i + batchSize);
    const result = await magfa.Send({
      recipients: batch,
      messages: message
    });

    results.push(result);
    console.log(`Batch ${i / batchSize + 1}: ${result.count.success} sent, ${result.count.fail} failed`);
  }

  return results;
}
```

### Custom Sender for Individual Messages

```typescript
// While the constructor sets a default sender,
// all messages will use this sender.
// To use different senders, create multiple Magfa instances.

const magfa1 = new Magfa('user', 'pass', 'domain', '3000111111');
const magfa2 = new Magfa('user', 'pass', 'domain', '3000222222');

await magfa1.Send({ recipients: '09123456789', messages: 'From first number' });
await magfa2.Send({ recipients: '09187654321', messages: 'From second number' });
```

### Polling Message Status

```typescript
async function waitForDelivery(mid: number, maxAttempts = 10): Promise<number> {
  for (let i = 0; i < maxAttempts; i++) {
    const result = await magfa.Statuses([mid]);
    const status = result.dlrs[0]?.status;

    if (status === 1) {
      console.log('Message delivered!');
      return status;
    } else if (status === 2 || status === 16) {
      console.log('Message delivery failed');
      return status;
    }

    // Wait 5 seconds before next check
    await new Promise(resolve => setTimeout(resolve, 5000));
  }

  console.log('Delivery status unknown after maximum attempts');
  return 0;
}

// Usage
const result = await magfa.Send({
  recipients: '09123456789',
  messages: 'Test'
});
const mid = result.messages[0].id;
await waitForDelivery(mid);
```

## Error Codes Reference

### Send Errors (1-31, 101-110)

| Code | Description |
|------|-------------|
| 1 | شماره گیرنده نادرست است |
| 2 | شماره فرستنده نادرست است |
| 3 | پارامتر encoding نامعتبر است |
| 4 | پارامتر mclass نامعتبر است |
| 6 | پارامتر UDH نامعتبر است |
| 13 | محتویات پیامک خالی است |
| 14 | مانده اعتبار کافی نیست |
| 15 | سرور مشغول است - ارسال مجدد |
| 16 | حساب غیرفعال است |
| 17 | حساب منقضی شده است |
| 18 | نام کاربری یا کلمه عبور نامعتبر است |
| 19 | درخواست معتبر نیست |
| 20 | شماره فرستنده به حساب تعلق ندارد |
| 22 | سرویس فعال نشده است |
| 23 | امکان پردازش وجود ندارد - تلاش مجدد |
| 24 | شناسه پیامک معتبر نیست |
| 25 | نام متد معتبر نیست |
| 27 | گیرنده در لیست سیاه است |
| 28 | گیرنده مسدود است |
| 29 | IP اجازه دسترسی ندارد |
| 30 | تعداد بخش‌ها بیش از حد مجاز است |
| 31 | داده‌های ناکافی |
| 101 | عدم تطابق طول آرایه messageBodies |
| 102 | عدم تطابق طول آرایه messageClass |
| 103 | عدم تطابق طول آرایه senderNumbers |
| 104 | عدم تطابق طول آرایه udhs |
| 105 | عدم تطابق طول آرایه priorities |
| 106 | آرایه گیرندگان خالی است |
| 107 | طول آرایه گیرندگان بیش از حد مجاز |
| 108 | آرایه فرستندگان خالی است |
| 109 | عدم تطابق طول آرایه encoding |
| 110 | عدم تطابق طول آرایه checkingMessageIds |

### Delivery Status Codes

| Code | Description |
|------|-------------|
| -1 | شناسه موجود نیست |
| 0 | وضعیتی دریافت نشده |
| 1 | رسیده به گوشی |
| 2 | نرسیده به گوشی |
| 8 | رسیده به مخابرات |
| 16 | نرسیده به مخابرات |

## Utility Functions

### GetStatusText

Get human-readable status message:

```typescript
import { GetStatusText } from 'magfa-api';

const statusText = GetStatusText(18);
console.log(statusText); // "نام کاربری و یا کلمه عبور نامعتبر است"
```

## TypeScript Support

This package is written in TypeScript and includes full type definitions. All types are exported and can be imported:

```typescript
import Magfa, {
  MagfaError,
  Encoding,
  ISentMessage,
  ISendResponseWithCount,
  GetStatusText,
  magfaErrors,
  messageStatuses
} from 'magfa-api';
```

You can also import types from the TypeScript source directly:

```typescript
import type { ISentMessage } from 'magfa-api/ts';
```

## Browser Support

While this package is primarily designed for Node.js, it uses the standard `fetch` API which is available in modern browsers. However, note that:

1. Browser security policies may block direct API calls (CORS)
2. Credentials would be exposed in client-side code
3. It's recommended to use this package on the server-side only

## License

MIT © [Shahab Movahhedi](https://shmovahhedi.com/)

## Resources

- [Official Magfa API Documentation](https://messaging.magfa.com/ui/public/wiki/api/http_v2)
- [GitHub Repository](https://github.com/movahhedi/magfa-api)
- [NPM Package](https://www.npmjs.com/package/magfa-api)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Author

**Shahab Movahhedi**
- Website: [shmovahhedi.com](https://shmovahhedi.com/)
- Email: dev@shmovahhedi.com

## Support

If you find this package helpful, please consider:
- ⭐ Starring the [GitHub repository](https://github.com/movahhedi/magfa-api)
- 🐛 Reporting bugs or suggesting features via [GitHub Issues](https://github.com/movahhedi/magfa-api/issues)
- 📖 Improving documentation
