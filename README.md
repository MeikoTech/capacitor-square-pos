# @meikotech/capacitor-square-pos

Integrate with Square Payments SDK

| Package Version | Capacitor Version |
| --------------- | ----------------- |
| 1.X             | 6.X               |

## Install

Version 1.X is compatible with Capacitor 6.X

```bash
npm install @meikotech/capacitor-square-pos
npx cap sync
```

## Usage

App Initalisation - App.ts (React example)

```ts
CapacitorSquare.initApp({
  applicationId: 'SQUARE-APP-ID',
})
  .then(() => {
    console.log('Square payment ready');
  })
  .catch((error) => {
    console.error(error.message);
  });
```

Payment flow

```ts
import { App } from '@capacitor/app';
import { CapacitorSquare } from '@meikotech/capacitor-square-pos';

// In Hook or component //

useEffect(() => {
  let appUrlListener, failedListener, successListener;

  const setupListeners = async () => {
    appUrlListener = App.addListener('appUrlOpen', (data) => {
      if (data.url.toLowerCase().startsWith('app-url-scheme://callback-url')) {
        CapacitorSquare.handleIosResponse({ url: data.url });
      }
    });

    failedListener = await CapacitorSquare.addListener('transactionFailed', handleFailedPayment);

    successListener = await CapacitorSquare.addListener('transactionComplete', (callback) => {
      handleSuccessPayment(callback);
    });
  };
  setupListeners();

  return () => {
    if (appUrlListener) appUrlListener.remove();
    if (failedListener) failedListener.remove();
    if (successListener) successListener.remove();
  };
}, [handleFailedPayment, handleSuccessPayment]);

const handleSquarePayment = (amount, orderId, squareLocationId) => {
  return new Promise((resolve, reject) => {
    CapacitorSquare.startTransaction({
      totalAmount: 100, // amount in pennies/cents
      currencyCode: 'CAD', // ISO currency code, must be support by square
      allowedPaymentMethods: ['CARD'], // Sqaure TendType: https://developer.squareup.com/docs/api/point-of-sale/android/com/squareup/sdk/pos/ChargeRequest.TenderType.html
      autoReturnTimeout: 4000, // The timeout to set in milliseconds, or AutoReturn.NoTimeout. If you specify a timeout, it must be between 3200 milliseconds and 10000 milliseconds.
      callbackUrl: 'app-url-scheme://callback-url', // see iOS setup
      note: 'Transaction note',
      locationId: 'SQ_location_id', // Use this to enforce payment for the correct business & location. Otherwise any logged in square account will work.
    })
      .then((result) => {
        console.log(result);
        resolve(result);
      })
      .catch((error) => {
        console.error(error);
        reject(error);
      });
  });
};
```

> Note: autoReturnTimeout is only available on Android

Follow these setup steps from square to enable call back to your app: [Square Documentation](https://developer.squareup.com/docs/pos-api/build-on-ios#step-4-add-your-url-schemes).
