## Exlpain

Formato de interpretación de newSchema

```js
{
    newKey:oldKey
}
```
## newSchema
```js
{
    purchaseOrder.orderNumber:orderId,
    destination.phone:destinationTelephoneNumber,
    destination.firstName:destinationFirstName,
    destination.lastName:destinationLastName,
    destination.email:destinationEmailAddress,
    purchaseOrder.expectedDate:orderExpectedDate,
    destination.name:destinationFacilityName,
    destination.code:originFacilityAliasId,
    destination.address:destinationAddress,
    destination.address.city:destinationCity,
    destination.address.country:destinationCountry,
    destination.address.postalCode:destinationPostalCode,
    purchaseOrder.totalAmount:monetaryValue,
    purchaseOrder.currency:mvCurrencyCode,
    vendor.code:originFacilityAliasId,
    destination.code:destinationFacilityAliasId,
    destination.address.state:destinationStateProv,
    destination.code:designatedCarrierCode,
    vendor.tributaryId:referenceField2,
    vendor.address.line1:originAddress,
    vendor.address.line2:originAddress2,
    vendor.address.line3:originAddress3,
    vendor.address:originAddress,
    vendor.name:originFacilityName,
    purchaseOrder.dispatchType:dispatchType,
    purchaseOrder.orderStatus:orderStatus,
    vendor.code:businessPartnerCode,
    purchaseOrder.createdDate:orderCreatedDate,
    purchaseOrder.expectedDate:orderExpectedDate,
    purchaseOrder.cancelDate:orderCancelDate,
    0:returnMiscellaneousNumber,
    purchaseOrder.orderType:sellType
}
```
### WarningSchema

- Algunas de las oldKeys no estan presentes en el json de prueba (orginal.json).
- Algunas de las oldKeys no se mapean en purchaseOrderMapper.js.
- Algunas de las oldKeys sobreescriben la newKey cambiando su valor.

###### [consular Issues y Warnings]


## Issues
1. destinationTelephoneNumber: El campo no existe en el json de prueba (original.json).
2. destinationFirstName: El campo no existe en el json de prueba (original.json).
3. destinationLastName: El campo no existe en el json de prueba (original.json).
4. destinationEmailAddress: El campo no existe en el json de prueba (original.json).
5. destinationPostalCode: El campo no existe en el json de prueba (original.json).
6. destinationFacilityAliasId: El campo usa la misma newKey (destination.code) que originFacilityAliasId, con lo cual se pisa el valor de la misma.
7. designatedCarrierCode: El campo no existe en el json de prueba (original.json). y ademas usa una newKey (destination.code) que ya esta en uso.
8. referenceField2: El campo no existe en el json de prueba (original.json).
9. originAddress: El campo no existe en el json de prueba (original.json).
10. originAddress2: El campo no existe en el json de prueba (original.json).
11. originAddress3: El campo no existe en el json de prueba (original.json).
12. businessPartnerCode: El campo usa una newKey (vendor.code) que ya esta en uso.
13. Se usa dos veces para escribir mismo campo. A continuación una muestra de como se utilizan:

### MESSAGE_FIELDS_MAP (odbmsMapper.js, bff).
```js
{
  deliveryDate: 'orderExpectedDate',
  orderExpectedDate: 'orderExpectedDate',
}
```
### return (purchaseOrderMapper.js, svl-fn-logistics).
```js
{
  orderExpectedDate: get(purchaseOrder, 'expectedDate'),
  deliveryDate: get(purchaseOrder, 'expectedDate'),
}
```
14. dispatchSuggestedDate: No se mapea en purchaseOrderMapper.
15. dispatchDefinitiveDate: No se mapea en purchaseOrderMapper.
16. budgetedCost: No se mapea en purchaseOrderMapper.
17. declaredValue: No se mapea en purchaseOrderMapper.
18. sellType: El campo no existe en el json de prueba (original.json).



## Warnings
1. destinationAddress: usa como newKey a destination.address, el cual es un objeto que purchaseOrderMapper.js destructura con getAddressLine(get(destination, 'address')).
2. returnMiscellaneousNumber: usa como newKey un cero. `0:returnMiscellaneousNumber`.

# purchaseOrderMapper.js
```js
// eslint-disable-next-line prettier/prettier
const { get, invert, toLower, map, mapValues, isEmpty, noop, keys, pick, pickBy, toArray, toUpper, trim, replace} = require('lodash');
const { BUSINESS_NAMES, CHILE, SELL_TYPE_MAPPER, SELL_TYPE_MARKETPLACE } = require('../config/constants');
const PRODUCT_MAP = require('./entities/product');
const DESTINATION_MAP = require('./entities/destination');
const PURCHASE_ORDER_MAP = require('./entities/purchaseOrder');
const VENDOR_MAP = require('./entities/vendor');
const ADDRESS_MAP = require('./entities/address');

const getBusinessIdbyName = name => invert(BUSINESS_NAMES)[toLower(name)];

const getAmountInCents = amount => Number.parseFloat(replace(amount, ',', '.')) * 100;

const getAddressLine = ({ line1, line2, line3 }) =>
  map(toArray(pickBy([line1, line2, line3])), item => trim(item)).join(', ');

const getData = (data, path, keysToGet) =>
  mapValues(pick(get(data, path), keys(keysToGet)), item => trim(item));

const getProducts = products =>
  map(products, product => ({
    ...mapValues(pick(product, keys(PRODUCT_MAP)), item => trim(item)),
    priceCents: getAmountInCents(trim(get(product, 'unitaryAmount'))),
    unitaryAmount: getAmountInCents(trim(get(product, 'unitaryAmount'))),
    totalAmount: getAmountInCents(trim(get(product, 'totalAmount')))
  }));

module.exports = data => {
  const header = get(data, 'header');
  const metadata = get(data, 'metadata');

  if (isEmpty(metadata) || isEmpty(header)) return noop();

  const purchaseOrder = getData(metadata, 'purchaseOrder', PURCHASE_ORDER_MAP);
  const destination = {
    ...getData(metadata, 'destination', DESTINATION_MAP),
    address: getData(metadata, 'destination.address', ADDRESS_MAP)
  };
  const vendor = {
    ...getData(metadata, 'vendor', VENDOR_MAP),
    address: getData(metadata, 'vendor.address', ADDRESS_MAP)
  };

  return {
    businessId: getBusinessIdbyName(get(header, 'commerce')),
    country: toUpper(trim(get(header, 'country', CHILE))),
    externalId: get(purchaseOrder, 'orderNumber'),
    currency: get(purchaseOrder, 'currency'),
    totalCents: getAmountInCents(get(purchaseOrder, 'totalAmount')),
    dispatchType: get(purchaseOrder, 'dispatchType'),
    orderStatus: get(purchaseOrder, 'orderStatus'),
    orderCreatedDate: get(purchaseOrder, 'createdDate'),
    orderExpectedDate: get(purchaseOrder, 'expectedDate'),
    orderCancelDate: get(purchaseOrder, 'cancelDate'),
    deliveryDate: get(purchaseOrder, 'expectedDate'),
    netCost: getAmountInCents(get(purchaseOrder, 'netCost')),
    tax: getAmountInCents(get(purchaseOrder, 'tax')),
    sellType: SELL_TYPE_MAPPER[get(purchaseOrder, 'orderType', SELL_TYPE_MARKETPLACE)],
    deliveryState: get(purchaseOrder, 'orderStatus'),
    paymentMethod: get(purchaseOrder, 'paymentType'),
    paymentState: get(purchaseOrder, 'paymentState', 'released'),
    customerFirstName: get(destination, 'firstName'),
    customerLastName: get(destination, 'lastName'),
    customerEmail: get(destination, 'email'),
    phonenumber: get(destination, 'phone'),
    deliveryName: get(destination, 'name'),
    destinationWarehouseId: get(destination, 'code'),
    deliveryAddress: getAddressLine(get(destination, 'address')),
    deliveryLocation: get(destination, 'address.location'),
    deliveryPostalCode: get(destination, 'address.postalCode'),
    deliveryCity: get(destination, 'address.city', 'test'),
    destinationStateOrProvince: get(destination, 'address.state'),
    deliveryCountry: get(destination, 'address.country'),
    deliveryCountryCode: get(destination, 'address.countryCode'),
    destinationFacilityAliasId: get(destination, 'code'),
    destinationCounty: get(destination, 'address.county'),
    carrierCode: get(destination, 'code'),
    companyId: get(vendor, 'code'),
    clientRut: get(vendor, 'tributaryId'),
    originAddressLine1: get(vendor, 'address.line1'),
    originAddressLine2: get(vendor, 'address.line2'),
    originAddressLine3: get(vendor, 'address.line3'),
    originLocation: get(vendor, 'address.location'),
    originCity: get(vendor, 'address.city'),
    originState: get(vendor, 'address.state'),
    originCountry: get(vendor, 'address.country'),
    originCountryCode: get(vendor, 'address.countryCode'),
    collectionAddress: getAddressLine(get(vendor, 'address')),
    store: get(vendor, 'name', 'test'),
    businessPartnerCode: get(vendor, 'code'),
    products: getProducts(get(metadata, 'products')),
    paymentAddressLine1: get(destination, 'address.line1'),
    deliveryCommune: get(destination, 'address.commune'),
    ivaCents: 0,
    householdOffice: 'test'
  };
};

```
# odbmsMapper.js


```js
const util = require('util');
const logger = require('../logger');
const odbmsLogger = require('../logger/external').odbms;
const {SODIMAC} = require('../config/constants');
const {getDateWithFormat} = require('../../server/helpers/formatDate');
const {SODIMAC_ORDER_STATUS_MAPPING, SODIMAC_WMOS_STORES_MAPPING,
  DATETIME_CL} = require('../config/constants');
const {get} = require('lodash');
const {sellTypeResolver} = require('../helpers/sellTypeHelper');


const MESSAGE_FIELDS_MAP = {
  externalId: 'orderId',
  phonenumber: 'destinationTelephoneNumber',
  customerFirstName: 'destinationFirstName',
  customerLastName: 'destinationLastName',
  customerEmail: 'destinationEmailAddress',
  deliveryDate: 'orderExpectedDate',
  deliveryName: 'destinationFacilityName',
  // TODO: Workarround field name for destinationWarehouseId
  destinationWarehouseId: 'originFacilityAliasId',
  deliveryAddress: 'destinationAddress',
  deliveryCity: 'destinationCity',
  deliveryCountry: 'destinationCountry',
  deliveryPostalCode: 'destinationPostalCode',
  totalCents: 'monetaryValue',
  currency: 'mvCurrencyCode',
  companyId: 'originFacilityAliasId',
  destinationFacilityAliasId: 'destinationFacilityAliasId',
  destinationStateOrProvince: 'destinationStateProv',
  carrierCode: 'designatedCarrierCode',
  clientRut: 'referenceField2',
  originAddressLine1: 'originAddress',
  originAddressLine2: 'originAddress2',
  originAddressLine3: 'originAddress3',
  collectionAddress: 'originAddress',
  store: 'originFacilityName',
  dispatchType: 'dispatchType',
  orderStatus: 'orderStatus',
  businessPartnerCode: 'businessPartnerCode',
  orderCreatedDate: 'orderCreatedDate',
  orderExpectedDate: 'orderExpectedDate',
  orderCancelDate: 'orderCancelDate',
  dispatchSuggestedDate: 'dispatchSuggestedDate',
  dispatchDefinitiveDate: 'dispatchDefinitiveDate',
  ivaCents: 'returnMiscellaneousNumber',
  budgetedCost: 'budgetedCost',
  declaredValue: 'declaredValue',
  sellType: 'sellType',
};

const mapper = (mapList, object, finder = (collection, key) => collection[key]) => (
  Object.keys(mapList).reduce((previous, localAttribute) => {
    return {...previous, ...{[localAttribute]: finder(object, mapList[localAttribute])}};
  }, {})
);

module.exports = data => {
  try {
    if (typeof data === 'object' && Object.keys(data).length === 0) return;
    const message = data.metadata;
    const products = message.products;
    let object = {};

    object = {...object, ...mapper(MESSAGE_FIELDS_MAP, message)};
    object['products'] = products.map(lineItem => {
      let product = {};
      product.quantity = parseInt(lineItem.quantity.orderQty);
      product.priceCents = parseFloat(lineItem.unitMonetaryValue) * 100;
      product.sku = lineItem.sku;
      product.description = lineItem.description;
      product.name = lineItem.description;
      product.ean = get(lineItem, 'commodityCode', null);
      product.sellerSku = get(lineItem, 'unNumber', null);

      return product;
    });
    object.businessId = SODIMAC;
    object.deliveryDate = getDateWithFormat(object.deliveryDate, DATETIME_CL);
    object.orderCreatedDate = getDateWithFormat(object.orderCreatedDate, DATETIME_CL);
    object.orderExpectedDate = getDateWithFormat(object.orderExpectedDate, DATETIME_CL);
    object.orderCancelDate = getDateWithFormat(object.orderCancelDate, DATETIME_CL);
    /* TODO: Currently we have status for Taken, Partial and Total Reception
    For Cancel and Rejected cases I put on closed until we have an status for canceled orders. */
    object.deliveryState = SODIMAC_ORDER_STATUS_MAPPING[object.orderStatus];
    object.store = SODIMAC_WMOS_STORES_MAPPING[object.destinationFacilityAliasId];
    object.collectionAddress = object.deliveryAddress;
    object.sellType = sellTypeResolver(SODIMAC, object.sellType);

    // TODO: Review this hardcoded values
    object.paymentAddressLine1 = 'test';
    object.deliveryCommune = 'test';
    object.originAddressLine1 = 'test';
    object.householdOffice = 'test';
    object.paymentState = 'released';
    object.paymentMethod = 'CMR';

    odbmsLogger.info('odbms succesfully parsed');
    return object;
  } catch (e) {
    logger.info(util.inspect(e, null, false));
  }
};

```




# Sources
###### Sodimac BFF copy by Felipe
- https://svl-team.postman.co/collections/1007594-13dd0254-aae6-4438-bacc-b92d6d8c2a19?version=latest&workspace=65347005-6f79-4fec-8731-f1f2d128d9c6#a4effa42-98b3-4b86-ab9a-458fec14e30d
- https://www.getpostman.com/collections/c8b51b3efeb4cdd7ebe2