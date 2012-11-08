node-screener
=============

Recursively screen and whitelists javascript objects with optional and flexible validation/processing of fields. Useful for filtering documents fetched by Mongoose in Node.JS and for any REST API.

Examples
=============

Specification for an object will be called a spec. Given a spec, screen will recursively
walk through the given object picking only fields specified and only if they pass
a given screener. You will find list of available screeners in the next section.

Suppose you have array of javascript objects that you fetched from a Mongo database:

  var object = [
    {
      "person": {
        "_id": "503cb6d92c32a8cd06006c53",
        "photoUrlId": "/user/503cb6d92c32a8cd06006c53.jpg",
        "user": {
          "description": "This is my user description"
          "name": "Joe Doe",
          "sex": "f",
          "birthdate": "04.07.1980",
        }
      },
      "occupancies": [
        {
          "_id": "503cb6d92c32a8cd06006c58",
          "servicePoint": {
            "_id": "503cb6d92c32a8cd06006c57",
            "address": [
              {
                "street": "Warszawska",
                "building": "56",
                "city": "Poznań",
                "postcode": "60-603",
              }
            ],
            "name": "Texi Drivers Limited.",
            "location": [
              16.9015636,
              52.3971881
            ]
          },
          "specialty": {
            "opis": "Taxi Driver - they move your butt from place to place",
            "name": "Taxi Driver",
            "_id": "4eed2d88c3dedf0d0300001c"
          }
        }
      ]
    }
  ];

Now let's cut some of the private unnecessary stuff with this simple example:

  var screen = require('screener').screen;
  var res = screen(object, [{
    person: {
      photoUrlId: true,
      user: {
        description: true,
        name: true,
        sex: true,
        birthdate: true,
      }
    },
    occupancies: [true],
  }]);
The resulting object looks like this:

  [{
    "person": {
      "photoUrlId": "/user/503cb6d92c32a8cd06006c53.jpg",
      "user": {
        "description": "This is my user description",
        "name": "Joe Doe",
        "sex": "f",
        "birthdate": "04.07.1980"
      }
    },
    "occupancies": [{
  //.. let's not paste the whole content .. but it would here all, believe me :)
      }
    }]
  }]
The "true" boolean screener simply fetches everything from the object at the given field.
Please note there here we also use an implicit "array" scanner by enclosing the spec in the array brackets([]).

What about validation of fields? How can we assure the content is what we expect to get?
Let's improve our example a little bit:

  // here is how you can register your own "screeners"
  // now you can reuse specification of a specific object
  screen.define('address', function(value) {
    return screen(value, {
      street: 'string',
      building: 'number',
      city: 'string',
      postcode: /\d\d-\d\d\d/
    });
  });

  function customFunctionValidator(value) {
    // just return undefined if the input is wrong
    // .. or modify te value to upper case should you wish!
    return value;
  }

  var res = screen(object, [{
    person: {
      photoUrlId: /user\/[a-f0-9]{16}\.jpg/,
      user: {
        description: customFunctionValidator,
        name: 'string',
        sex: /[fm]/,
        birthdate: /\d\d\.\d\d\.\d\d\d\d/,
      }
    },
    occupancies: [{
      servicePoint: {
        address: 'address'
      }
    }],
  }]);

In the above example we made use of regexp screeners which approve the content that fully matches the regexp (/a/ will not match "abc"). We have also demonstrated the use of custom validators. Now you hopefully understand the basics of this module. ;)

Should you want to do more complex stuff, see the below quick examples:

  var mergeTest = {address: {street: "Słowackiego", building: 5}};

  var res = screen(mergeTest, {address: screen.merge({street: 'string', building: 'number'})});

And the result is:

  {street: "Słowackiego", building: 5}

With this method you can easily flatten your objects. There is also `screen.or` and `screen.and` which do logical operation on screeners given as consequent arguments.