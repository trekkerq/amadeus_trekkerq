# amadeus_trekkerq
Flight booking app using Node and Vue

Setting up a simple Node.js server
The first thing you’ll need to do is set up a simple Node.js server. Here’s an example of a minimal express server listening on port 2800:

var express = require('express') 
var http = require('http').createServer(app);
var app = express(), 
var server = app.listen(process.env.PORT || 2800,()=>{
  console.log("Howdy, I am running at PORT 2800")
})

Next, install the Amadeus Node package:

 


npm install amadeus --save
 

Then plug in to our server using our credentials :

 


var Amadeus = require('amadeus');
var amadeus = new Amadeus({
  clientId: 'REPLACE_WITH_YOUR_API_KEY',
  clientSecret: 'REPLACE_WITH_YOUR_API_SECRET'
});
 

You can test the server by connecting to localhost:2800 from your browser. The server is currently very basic but you'll be expanding it later.

 

 

Getting a list of cities
As mentioned above, this app has an airport/city search functionality. To learn more about how to set up a server for city search, read this article on airport autocomplete using the MERN stack.

For this tutorial, you can pass the following code directly into the URL localhost:2800/citysearch?keyword=“YOUR_SEARCH”.

 


app.get(`/citySearch`, async (req, res) => { 
  console.log(req.query); 
  var keywords = req.query.keyword; 
  const response = await amadeus.referenceData.locations 
    .get({ 
      keyword: keywords, 
      subType: "CITY,AIRPORT", 
    }) 
    .catch((x) => console.log(x)); 
  try { 
    await res.json(JSON.parse(response.body)); 
  } catch (err) { 
    await res.json(err); 
  } 
});
 

 

Creating a POST request to search flight deals
The first step in building a booking engine is performing a flight search with Flight Offers Search. You’ll need to call the endpoint /shopping/flight-offers.

Here is how to build the request on the backend:

 


app.post("/date", async function (req, res) { 
  console.log(req.body); 
  departure = req.body.departure; 
  arrival = req.body.arrival; 
  locationDeparture = req.body.locationDeparture; 
  locationArrival = req.body.locationArrival; 
  const response = await amadeus.shopping.flightOffersSearch 
    .get({ 
      originLocationCode: locationDeparture, 
      destinationLocationCode: locationArrival, 
      departureDate: departure, 
      adults: "1", 
    }) 
    .catch((err) => console.log(err)); 
  try { 
    await res.json(JSON.parse(response.body)); 
  } catch (err) { 
    await res.json(err); 
  } 
}); 
 


You can now call http://localhost:2800/date?departure=2020-05-01&arrival=2020-02-27&locationDeparture=MAD&locationArrival=LAX to get a list of flight-offers objects. Remember that you can modify these parameters.

Now that you have your flight offers, the next step is to confirm the final price.
 

 

Calling Flight Offers Price to get the final flight price
The price and availability of airfare can change between the time of search and the time of booking, so you’ll need to confirm the final price using Fight Offers Price before sending it to the Create Order endpoint. To do this, select an item from Flight Offers Search response and pass it into the inputFlight variable:

 


app.post('/flightprice', async function(req, res) {
  res.json(req.body);
  inputFlight = req.body;
  console.log(req.body)
  const responsePricing = await amadeus.shopping.flightOffers.pricing.post(
      JSON.stringify({
        'data': {
          'type': 'flight-offers-pricing',
          'flightOffers': inputFlight
        }})).catch(err=>console.log(err))
   try {
    await res.json(JSON.parse(responsePricing.body));
  } catch (err) {
    await res.json(err);
  }
   })
 

The API will return the final price of the selected flight. Once this is done, you’re ready to book your ticket!

 

 

Calling Flight Create Orders to complete the booking
To complete the booking using Flight Create Orders, create a request containing the selected flight offer and the passenger data (name, lastname and mail are sufficient for our purposes) and pass it to a function. Here, inputFlightCreateOrder is the response from Flight Offers Price:

 


app.post('/flightCreateOrder', async function(req, res) {
  res.json(req.body);
  let inputFlightCreateOrder = req.body;
console.log(req.body)
const returnBokkin = amadeus.booking.flightOrders.post(
      JSON.stringify({
  "data": {
    "type": "flight-order",
    "flightOffers": [
           inputFlightCreateOrder
        ],
    "travelers": [
      {
        "id": "1",
        "dateOfBirth": "1982-01-16",
        "name": {
          "firstName": "JORGE",
          "lastName": "GONZALES"
        },
        "gender": "MALE",
        "contact": {
          "emailAddress": "jorge.gonzales833@telefonica.es",
          "phones": [
            {
              "deviceType": "MOBILE",
              "countryCallingCode": "34",
              "number": "480080076"
            }
          ]
        },
        "documents": [
          {
            "documentType": "PASSPORT",
            "birthPlace": "Madrid",
            "issuanceLocation": "Madrid",
            "issuanceDate": "2015-04-14",
            "number": "00000000",
            "expiryDate": "2025-04-14",
            "issuanceCountry": "ES",
            "validityCountry": "ES",
            "nationality": "ES",
            "holder": true
          }
        ]
      },
      {
        "id": "2",
        "dateOfBirth": "2012-10-11",
        "gender": "FEMALE",
        "contact": {
          "emailAddress": "jorge.gonzales833@telefonica.es",
          "phones": [
            {
              "deviceType": "MOBILE",
              "countryCallingCode": "34",
              "number": "480080076"
            }
          ]
        },
        "name": {
          "firstName": "ADRIANA",
          "lastName": "GONZALES"
        }
      }
    ]
  }
})
    ).then(function(response){
    console.log(response.result);
}).catch(function(responseError){
    console.log(responseError);
});
})
 

In the response object, there is a value call reference with the flight confirmation number.

Congrats! You are ready to fly!

 

 

Intermission and source code
This marks the end of Part 1. So far, you've used the Amadeus Node SDK to set up a basic Node.js server and start calling the three APIs that make up the flight booking flow. In Part 2, you'll complete your flight booking app by building the Vue.js frontend to let users search, select and book their flights.

The complete code for Part 1 is available on GitHub:  https://github.com/amadeus4dev/amadeus-flight-booking-node

To run the server, switch to the server folder and install the dependencies with:

 

npm  install

 

Then run the server by typing:

 

npm run start

 

The backend will listen for an incoming connection on port 2800. At this point you can either use the Vue.js client in the repository or Postman to perform the calls to the backend.
