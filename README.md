# SDK Tranzzo Tokenizer

[![License]
[![Platform](https://img.shields.io/badge/android-platform-green)](https://github.com/tranzzo/ios-card-tokenizer/blob/dev/LICENSE)

## Requirements 
API 16+

## Instalation 
#### Gradle
```groovy
// Add Dagger dependencies
dependencies {
  api 'com.google.dagger:dagger:2.x'
  annotationProcessor 'com.google.dagger:dagger-compiler:2.x'
}
```


## Features

**Tokenization**: Simple way to obtain a Tranzzo user card token, generated on our servers.
    For more information visit our [**documentation**](https://cdn.tranzzo.com/tranzzo-api/index.html)
**Validation**: Use straightforward card utilities to validate, define and format your card input

## Usage 
1. Add Tranzzo views in your android project:
```groovy
// Add view for get data number card
<com.tranzzo.android.sdk.view.CardNumberEditText
.../>

// Add view for get date validate card
<com.tranzzo.android.sdk.view.ExpiryDateEditText
.../>

// Add view get cvc
<com.tranzzo.android.sdk.view.CvcEditText
.../>
```
 2. After bind viws in code create listener for data input:
 ```groovy
  cardInputListener = new TranzzoInputListener(
                etCardNumber,
                etExpiration,
                etCvc,
                () -> viewBtnTokenize.setEnabled(true)
        );
```
3. Create methods for collect data:
```groovy
   private Either<TrzError, Card> collectCard() {
        return etCardNumber
                .getCardNumber()
                .flatMap(cardNumber ->
                        etExpiration
                                .getValidDateFields()
                                .flatMap(expiry ->
                                        etCvc
                                                .getCvc()
                                                .map(cvc -> new Card(cardNumber, expiry, cvc))
                                )
                )
                .mapLeft(TrzError::mkInternal);
    }
```
4. Validate inputed data:
```groovy
  if (cardInputListener.isFormValid()) {
                collectCard().consume(
                        this::displayError,
                        c -> {
                            new TokenizeTask().execute(c);
                        }
                );
```
5. Execute request in background for get token:
```groovy
 private class TokenizeTask extends AsyncTask<Card, Void, Either<TrzError, CardToken>> {
        
        @Override
        protected Either<TrzError, CardToken> doInBackground(Card... cards) {
            return initTranzzo().tokenize(cards[0], getApplicationContext());
        }
        
        @Override
        protected void onPostExecute(Either<TrzError, CardToken> result) {
            result.consume(
                    TranzzoDemoActivity.this::displayError,
                    token -> {
                        displayResult(token.toString());
                        Log.i("TOKEN", ">>> " + token.toString());
                    });
        }
    }
```

