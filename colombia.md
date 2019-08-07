## Formato original
```js
{
  "message" : {
      "header": {
        "channel": "ODBMS",
        "commerce": "SODIMAC",
        "country": "CO",
        "datetime": "2019-04-03T16:27:45",
        "entityType": "purchaseOrder",
        "messageId": "6596A137-932E-4559-B480-DA92402BBD8F",
        "mimeType": "application/json",
        "timestamp": "20190403162745",
        "version": "1.0"
      },
      "metadata": {
        "purchaseOrder": {
          "orderNumber": "6788901823840256",
          "orderType": "MKP",
          "dispatchType": "direct_with_courier",
          "orderStatus": "readyToDispatch", // estado previo "awaitingCourierLabel"
        },
        "additionalAttributes": {
          "pickup": {
            "address": "avenida siempre viva",
            "pickupDate": "2019-04-03T16:27:45",
            "docs": [{
              "type": "dispatchGuide",
              "url": "http://document.pdf"
            }]
          }
        }
      }
    }
}
```

## jsonColombia

```js
{
    "Pvi_Orden_Compra": "8401560",
    "Pvi_Id_Detalle": "35514500,35514501,35514502",
    "Pvi_Url": "http://200.69.100.66/2impresionGuiaspruebas/Guia3.aspx?Usuario=EMPCAR01&Guia=014994716694",
    "Pvi_Guia": "014994716694",
    "Pvi_Orden_Servicio": "01-1795081",
    "Pvi_Url_Rotulos": "http://200.69.100.66/2IMPRESIONGUIASpruebas/ISticker_ZEA.aspx?Guia=014994716694",
    "Pvi_Fecha_Generacion": "Date(1565017877686)"
}
```


## Propuesta de estructura final

```
{
    "message": {
        "header": {
            "channel": "ODBMS",
            "commerce": "SODIMAC",
            "country": "CO",
            "datetime": "2019-04-03T16:27:45",
            "entityType": "purchaseOrder",
            "messageId": "6596A137-932E-4559-B480-DA92402BBD8F",
            "mimeType": "application/json",
            "timestamp": "20190403162745",
            "version": "1.0"
        },
        "metadata": {
            "purchaseOrder": {
                "orderNumber": "6788901823840256",
                "orderType": "MKP",
                "dispatchType": "direct_with_courier",
                "orderStatus": "readyToDispatch",
                "trackingNumber": "014994716694",
            },
            
            "additionalAttributes": {
                "pickup": {
                    "address": "avenida siempre viva",
                    "pickupDate": "2019-04-03T16:27:45",
                    "docs": [
                        {
                            "type": "dispatchGuide",
                            "url": "dispatchGuideUrl"
                        },
                        {
                            "type": "label",
                            "url": "labelUrl"
                        }
                    ]
                },
                "courrierName":"CORREOSDECHILE",
                "courrierCode": "01-1795081",
            }
        }
    }
}
```

# Cambios

```js
{
    "Pvi_Orden_Compra": "orderNumber", 
    "Pvi_Id_Detalle": "idDetail"
    "Pvi_Url": "pviUrl",
    "Pvi_Guia": "trackingNumber",
    "Pvi_Orden_Servicio": "courrierCode",
    "Pvi_Url_Rotulos": "pviRotulosUrl",
    "Pvi_Fecha_Generacion": "guideGenerationDate"
}

```

# Campos descartados
<del> "Pvi_Id_Detalle": "idDetail"  
<del> Pvi_Fecha_Generacion": "guideGenerationDate,

# Consideraciones

- Pvi_Guia: es el tracking number.
- courrierCode es un identificador numerico del courrier.
- Las guias de despacho y etiquetas deben ir como un objeto dentro de docs (array de objetos), indicando el tipo y url.
