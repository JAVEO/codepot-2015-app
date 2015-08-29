# Codepot Agenda Application

## Star rating

#### 1. Dodaj zależność do iron-localstorage
```
bower install --save PolymerElements/iron-localstorage
```
Opcja '--save' wpisuje zależność do bower.json. Na pytanie czy dodać element do elements.html odpowiedz twierdząco.


I dodaj do elements.html:
```
<link rel="import" href="../bower_components/iron-localstorage/iron-localstorage.html">
```

#### 2. Zależność do star-rating już jest (bo element ma błąd, więc do wrzuciliśmy do naszego repo wersję poprawioną). Ale jakby nie było, to:
```
bower install --save star-rating
```

#### 3. Musimy dodać obsługę zapisu do localstorage i do firebase, więc dodajemy wrapper na star-rating:
```
yo polymer:el my-star-rating
```

Jeżeli ktoś nie ma 'yo', to:
```
npm install -g generator-polymer
```

#### 4. W elemencie my-star-rating dodajemy star-rating i uzupełniamy properies:
```
<template>
	<star-rating rate="{{rate}}" on-tap="sendRateEvent"></star-rating>
<template>

<script>
(function () {
    Polymer({
        is: 'my-star-rating',
        properties: {
            rate: Number,
            workshop: String
        }
    });
})();
</script>
```

#### 5. Dodajemy obsługę ocenienia (kliknicie kompnonetu powoduje wysłanie eventu):
```
<script>
        (function () {
            Polymer({
                is: 'my-star-rating',
                properties: {
                    rate: Number,
                    workshop: String
                },
                sendRateEvent: function () {
                    this.fire('rate-changed', {workshop: this.workshop, rate: this.rate});
                }
            });
        })();
    </script>
```


#### 6. Dodajemy obsługę localstorage:
```
<template>
	<iron-localstorage name="{{localStorageKey}}" value="{{rate}}"></iron-localstorage>
	<star-rating rate="{{rate}}" on-tap="sendRateEvent"></star-rating>
<template>

<script>
        (function () {
            Polymer({
                is: 'my-star-rating',
                properties: {
                    rate: Number,
                    workshop: String,
                    localStorageKey: {
                        type: String,
                        computed: 'createLocalStorageKey(workshop)'
                    }
                },
                sendRateEvent: function () {
                    this.fire('rate-changed', {workshop: this.workshop, rate: this.rate});
                },
                createLocalStorageKey: function (workshop) {
                    return 'workshop-review-' + workshop.title;
                }
            });
        })();
    </script>
```

#### 7. Dodajemy element do workshop-list:
```
 <template is="dom-repeat" items="{{filtered}}" as="workshop">
    <paper-material elevation="3">
        (...)
        <my-star-rating on-rate-changed="addrate" workshop="{{workshop}}"></my-star-rating>
    </paper-material>
</template>

<script>
        Polymer({
            is: "workshops-list",
            addrate: function (event) {
                var workshopIndex = this.titleToId[event.detail.workshop.title];
                var rates = this.workshops[workshopIndex].rates;
                if (!rates) rates = [];
                rates.push(event.detail.rate);
                this.set('workshops.' + workshopIndex + '.rates', rates);
            },
            (...)
            extractIds: function (workshops) {
                var titles = {};
                for (var i in workshops) {
                    titles[workshops[i].title] = i ;
                }
                return titles;
            },
            properties: {
              (...)
              titleToId: {
                    type: Object,
                    computed: 'extractIds(workshops)'
                },
             (...)
            }
        });
    </script>
          
          
```

