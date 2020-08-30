---
layout: post
title:  "¿Cómo ver en Fintual los bitcoins en tu billetera de Buda?"
date:   2020-08-29 04:13:01 -0400
categories: tampermonkey
author: boris-macias
---
## ¿Buda?

Por si no conoces Buda, Buda es el mejor exchange de criptomonedas de sudamérica y <a href="https://www.buda.com" target="_blank"><b>acá</b></a> puedes ver más sobre ellos.

## ¿Fintual?

Bueno, si no conocieras Fintual no estarías acá.

## ¿Buda + Fintual?

Si, es posible, por ahora como plugin de Tampermonkey/Greasemonkey.

Antes de empezar este tutorial necesitas crear tu cuenta en <a href="https://www.buda.com" target="_blank"><b>Buda</b></a> y ojalá tener algunos satoshis en tu wallet (para poder ver que funciona el script).

Con tu cuenta creada en buda (y tus satoshis listos 🚀) tendrás que crear tu API Key y API Secret acá <a target="_blank" href="https://www.buda.com/manejar_api_keys">https://www.buda.com/manejar_api_keys</a>. Ponle un nombre a tu key, una fecha de expiración con la que te sientas cómod@ y <b>no actives las casillas "¿Puede crear órdenes?" ni "¿Puede hacer retiros?"</b> porque solo necesitamos las keys para ver tu saldo.

<img src="/assets/buda-fintual/api-key.png" style="width: 50%">

Guarda la llave y el secreto que se generarán en un lugar seguro (los vamos a necesitar más adelante).

Con tu cuenta Buda lista, ahora tenemos que [descargar](https://www.tampermonkey.net/) e instalar Tampermonkey

<img src="/assets/buda-fintual/tampermonkey-download.png" style="width: 50%">

Luego hacemos click en el plugin

<img src="/assets/buda-fintual/plugin.png" style="width: 50%">

Y en el menú que se despliega en "Create new script"

<img src="/assets/buda-fintual/create-script.png" style="width: 50%">

Se abrirá un editor en una nueva pestaña, donde tendrás que pegar el código que compone este plugin.<br>
Esta es la URL del código: <a target="_blank" href="https://raw.githubusercontent.com/borismacias/buda-fintual-tampermonkey/master/main.js">https://raw.githubusercontent.com/borismacias/buda-fintual-tampermonkey/master/main.js</a> (Al final de este artículo está la explicación paso a paso de lo que hace el código)

<img src="/assets/buda-fintual/raw-code.png" style="width: 50%">

Después de copiar y pegar el código, tienes que ir a "File" -> "Save"

<img src="/assets/buda-fintual/save.png">

Y listo, ya está instalado el plugin y podrás verlo cuando entres a la página principal de fintual, con esta URL exacta <a href="https://fintual.cl/" target="_blank">https://fintual.cl/</a>.

Ahora que ya tenemos el plugin instalado, la primera vez que entres a Fintual tendrás que configurarlo.

Primero aparece este cuadro de diálogo, acá tenemos que ingresar un texto al azar que se va a utilizar para poder encriptar tu API Key y API secret de Buda

<img src="/assets/buda-fintual/setup-1.png" style="width: 50%">

Despues tienes que ingresar la API Key de buda que generamos al principio de esta guía

<img src="/assets/buda-fintual/setup-2.png" style="width: 50%">

Y finalmente ingresa el API Secret.

<img src="/assets/buda-fintual/setup-3.png" style="width: 50%">

El API Key y el Secret solo se guardan localmente y además encriptados con el texto que ingresaste al comienzo de la configuración.

Ahora, si entras a tu cuenta Fintual, y vas a <a href="https://fintual.cl/" target="_blank">https://fintual.cl/</a> (no va a funcionar en ninguna otra URL) se abrirá un nuevo tab de Tampermonkey pidiéndote permisos para permitir que el script hable con otro dominio, en este caso necesita hablar con <b>https://buda.com</b>. Solo necesitas hacer click en <b>Always allow domain</b>

<img src="/assets/buda-fintual/setup-4.png" style="width: 50%">

Despues de <u>aprobar</u> los permisos, recarga la pestaña en la que estabas y deberías ver tu saldo de bitcoins en pesos bajo el título "Otras Platas" así

<img src="/assets/buda-fintual/setup-5.png" style="width: 50%">

Si por algún motivo el texto se queda pegado en "Hablando con Buda", reinicia Chrome y recarga la página

<b>Para cualquier sugerencia/corrección, pueden mandarme un PR a <a href="https://github.com/borismacias/buda-fintual-tampermonkey">https://github.com/borismacias/buda-fintual-tampermonkey</a> o un correo a <a href="mailto:boris@fintual.com">boris@fintual.com</a></b>

## El código

```
// @match        https://fintual.cl/
// @require      http://crypto.stanford.edu/sjcl/sjcl.js
// @require      https://cdnjs.cloudflare.com/ajax/libs/js-sha512/0.8.0/sha512.min.js
// @grant    GM_getValue
// @grant    GM_setValue
// @grant    GM_registerMenuCommand
// @grant    GM.xmlHttpRequest
```

`@match`: La URL donde este plugin se ejecuta, en este caso es https://fintual.cl/ <br>
`@require`: Se utiliza para cargar librerías exteras. En este script necesitamos la Stanford Javascript Crypto Library y sha512, para encriptar las credenciales y para encriptar la firma para usar la api de buda<br>
`@grant`: La usamos para "importar" funciones de greasemoney/tampermonkey

<br>

```
function buildHTML(capital) {
  return '<div class="fintual-app-goal-item">' +
           '<a class="fintual-app-goal-item__link" href="#"></a>' +
           '<div class="fintual-app-goal-item__details">' +
             '<div class="fintual-app-goal-item__name fintual-app-goal-item__name--deposited">' +
               '<span>Bitcoins ₿</span>' +
             '</div>' +
           '</div>' +
           '<div class="fintual-app-goal-item__status-detail">' +
             '<div class="fintual-app-goal-item__amount">'+ capital +'</div>' +
           '</div>' +
         '</div>'
}
```
`buildHTML(capital)`: Construye los nodos que muestran el monto en CLP

<br>

```
function checkEncKey() {
  let encKey = GM_getValue('encKey', '');

  if (!encKey) {
    encKey = prompt(
      'Necesitamos un texto random para poder encriptar tus datos. Porfa ingresa cualquier texto:',
    );
    GM_setValue('encKey', encKey);
  }
}
```
`checkEncKey()`: Revisa si existe el valor para la llave que encripta las credenciales

<br>

```
function buildMenuCommands() {
  GM_registerMenuCommand('Cambiar API Key', changeApiKey);
  GM_registerMenuCommand('Cambiar API Secret', changeApiSecret);
}

function changeApiKey() {
  promptAndChangeStoredValue('Api key', 'apiKey');
  rebuildBitcoinContainerInnerHtml('Recarga la página 🙏🏼')
}

function changeApiSecret() {
  promptAndChangeStoredValue('Api secret', 'apiSecret');
  rebuildBitcoinContainerInnerHtml('Recarga la página 🙏🏼')
}

function promptAndChangeStoredValue(userPrompt, setValVarName) {
  let targVar = prompt('Cambiar ' + userPrompt, '');
  GM_setValue(setValVarName, encryptAndStore(targVar));
}

function rebuildBitcoinContainerInnerHtml(amount) {
  let bitcoinContainer = document.getElementById('bitcoin-container');
  bitcoinContainer.innerHTML = buildHTML(amount)
}
```

`buildMenuCommands()`: Crea menúes para poder cambiar manualmente la api key y api secret sin tener que desinstalar el plugin <br>
`changeApiKey()` - `changeApiSecret()`: Prompts y setters para ApiKey y ApiSecret<br>
`promptAndChangeStoredValue`: self explanatory<br>
`rebuildBitcoinContainerInnerHtml(amount)`: reconstruye el contenedor con el monto

<br>

```
function decodeOrPrompt(targVar, userPrompt, setValVarName) {
  if (!targVar) {
    targVar = prompt(userPrompt + ' no está seteada. Ingrésala acá:','');
    GM_setValue(setValVarName, encryptAndStore(targVar));
  }
  else {
    targVar = unStoreAndDecrypt(targVar);
  }
  return targVar;
}

function encryptAndStore(clearText) {
  const encKey = GM_getValue('encKey', '');
  return JSON.stringify(sjcl.encrypt(encKey, clearText));
}

function unStoreAndDecrypt(jsonObj) {
  const encKey = GM_getValue('encKey', '');
  return sjcl.decrypt(encKey, JSON.parse(jsonObj));
}
```
`decodeOrPrompt(targVar, userPrompt, setValVarName)`: retorna `tarVar` desencriptada si existe, sino usa un prompt con el texto `userPrompt` y la guarda en `setValVarName`<br>
`encryptAndStore(clearText)`: encripta el texto plano `clearText` usando la librería de stanford <br>
`unStoreAndDecrypt(jsonObj)`: retorna `jsonObj` desencriptado

<br>

```
function fillCapital(key, secret){
  const nonce = getNonce();
  const baseURL = 'https://www.buda.com';
  const balancePath = '/api/v2/balances';
  const unsignedString = 'GET ' + balancePath + ' ' + nonce;
  const signature = sha384.hmac(secret, unsignedString);
  const balanceUrl = baseURL + balancePath;

  GM.xmlHttpRequest({
    method: "GET",
    url: balanceUrl,
    headers: {
      'X-SBTC-APIKEY': key,
      'X-SBTC-SIGNATURE': signature,
      'X-SBTC-NONCE': nonce
    },
    onload: function(balanceResponse) {
      const parsedBalanceResponse = JSON.parse(balanceResponse.responseText);
      const btcBalance = parsedBalanceResponse.balances.find(function(balance){ return balance.id === "BTC"});
      const btcAmount = parseFloat(btcBalance.available_amount[0]);

      GM.xmlHttpRequest({
        method: "POST",
        headers: {
            'Accept': 'application/json',
            "Content-Type": "application/json"
        },
        url: "https://www.buda.com/api/v2/markets/btc-clp/quotations",
        data: JSON.stringify({type: "bid_given_earned_base",amount: btcAmount + ""}),
        dataType: 'json',
        contentType: 'application/json',
        overrideMimeType: 'application/json',
        onload: function (quotationResponse) {
          const parsedQuotationResponse = JSON.parse(quotationResponse.responseText);
          const capital = parsedQuotationResponse.quotation.quote_exchanged[0];
          rebuildBitcoinContainerInnerHtml(currencyFormattedAmount(capital))
        }
      })
    }
  });
}

function currencyFormattedAmount(amount, decimalCount = 0, decimal = ",", thousands = ".") {
  try {
    decimalCount = Math.abs(decimalCount);
    decimalCount = isNaN(decimalCount) ? 2 : decimalCount;

    const negativeSign = amount < 0 ? "-" : "";

    let i = parseInt(amount = Math.abs(Number(amount) || 0).toFixed(decimalCount)).toString();
    let j = (i.length > 3) ? i.length % 3 : 0;

    return '$ ' + negativeSign + (j ? i.substr(0, j) + thousands : '') + i.substr(j).replace(/(\d{3})(?=\d)/g, "$1" + thousands) + (decimalCount ? decimal + Math.abs(amount - i).toFixed(decimalCount).slice(2) : "");
  } catch (e) {
    console.log(e)
  }
}
```

`fillCapital(key, secret)`: Hace las llamadas a la API de buda para conseguir el balance de bitcoins y para calcular la equivalencia en CLP. Necesita la `key` y `secret`<br>
`currencyFormattedAmount(amount, decimalCount, decimal)`: Formatea el `amount` y retorna elmonto con `$` y `.`<br>
`getNonce()`: Retorna el timestamp de `now` para usarlo en las llamadas a la API de Buda<br>

<br>

```
(function() {
    'use strict';

    checkEncKey();
    let key = GM_getValue('apiKey', '');
    let secret = GM_getValue('apiSecret', '');
    key = decodeOrPrompt(key, 'ApiKey de Buda', 'apiKey');
    secret = decodeOrPrompt(secret, 'ApiSecret de Buda', 'apiSecret');
    if(!key || !secret){
      alert("Necesitamos tu key y secret para poder mostrar tu balance.")
    };
    buildMenuCommands();
    let goalsContainers = document.getElementsByClassName('fintual-app-goal-items');
    let base = goalsContainers[goalsContainers.length - 1];
    let bitcoinContainer = document.createElement('div');
    bitcoinContainer.setAttribute('id','bitcoin-container');
    base.append(bitcoinContainer);
    bitcoinContainer.innerHTML = buildHTML("Hablando con Buda ...");
    fillCapital(key, secret);
})();
```

Flujo principal del script
* Revisa que exista la llave para encriptar las credenciales
* Obtiene los valores para el API key + secret
* Si existen los valores, se desencriptan con la llave, sino se piden con un prompt
* Construye los menúes para cambiar la API key y el secret
* Construye los nodos HTML para escribir el monto
* Llama a `fillCapital`


