# Christchurch Bins integration with Home Assistant 

Christchurch [New Zealand] City Council has a great website for checking what bins need to go out for collection

![image-20211127165316748](https://raw.githubusercontent.com/durankeeley/Christchurch-Bins-with-Home-Assistant/main/assets/image-20211127165316748.png)


## Setting up the Data for Home Assistant

To integrate with Home Assistant you need to find out the property ID of your address 

Using the information on the lookup.js

```
https://ccc.govt.nz/resources/ccc-kerbside/client/dist/js/lookuptool.js
```

![image-20211127170317178](https://raw.githubusercontent.com/durankeeley/Christchurch-Bins-with-Home-Assistant/main/assets/image-20211127170317178.png)

You can use the rest URL followed by your house number + street address 

```
https://opendata.ccc.govt.nz/CCCSearch/rest/address/suggest?q=
```

Example:

```
https://opendata.ccc.govt.nz/CCCSearch/rest/address/suggest?q=53+Hereford
```

you will need the "RatingUnitID"

![image-20211127170041061](https://raw.githubusercontent.com/durankeeley/Christchurch-Bins-with-Home-Assistant/main/assets/image-20211127170041061.png)

Now using the information on the lookup.js again all we need to do is use the original website with "/getProperty?ID=[RatingUnitID]":

![image-20211127170759542](https://raw.githubusercontent.com/durankeeley/Christchurch-Bins-with-Home-Assistant/main/assets/image-20211127170759542.png)

![image-20211127170831635](https://raw.githubusercontent.com/durankeeley/Christchurch-Bins-with-Home-Assistant/main/assets/image-20211127170831635.png)

Example:

```
https://ccc.govt.nz/services/rubbish-and-recycling/collections/getProperty?ID=86089
```



That will give you a JSON output of that property:

![image-20211127171207369](https://raw.githubusercontent.com/durankeeley/Christchurch-Bins-with-Home-Assistant/main/assets/image-20211127171207369.png)

## Home Assistant

We need to add the scraping from the API into the configuration.yaml and some logic to get the bin of the week for lovelace

```yaml
sensor:
  - platform: rest
    resource: https://ccc.govt.nz/services/rubbish-and-recycling/collections/getProperty?ID=86089
    method: GET
    name: "Christchurch Bin Type"
    value_template: >
      {% set value_json_sort = value_json.bins.collections | sort(attribute='next_planned_date') | rejectattr('next_planned_date', 'le', (now().strftime('%Y-%m-%d')|string)) %}
      {% set value_json = value_json_sort | rejectattr('material', 'equalto', 'Organic') | map(attribute='material') | list | first %}
      {{ value_json}}
    scan_interval: 43200
  - platform: rest
    resource: https://ccc.govt.nz/services/rubbish-and-recycling/collections/getProperty?ID=86089
    method: GET
    name: "Christchurch Bin Date"
    value_template: >
      {% set value_json_sort = value_json.bins.collections | sort(attribute='next_planned_date') %}
      {% set value_json = value_json_sort | rejectattr('next_planned_date', 'le', (now().strftime('%Y-%m-%d')|string)) | map(attribute='next_planned_date')| list| first %}
      {{ value_json }}
    scan_interval: 43200
```

Then add a custom card to lovelace:

```yaml
type: vertical-stack
cards:
  - type: custom:button-card
    entity: sensor.christchurch_bin_type
    show_entity_picture: true
    state:
      - entity_picture: https://ccc.govt.nz/resources/ccc-kerbside/client/images/yellowbin.png
        value: Recycle
      - entity_picture: https://ccc.govt.nz/resources/ccc-kerbside/client/images/redbin.png
        value: Garbage
  - type: entity
    entity: sensor.christchurch_bin_date
    state_color: false
    icon: mdi:delete-clock
```

![image-20211127173015400](https://raw.githubusercontent.com/durankeeley/Christchurch-Bins-with-Home-Assistant/main/assets/image-20211127173015400.png)
