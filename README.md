# Cineplexed Documentation

[Play Cineplexed Here!](https://d3151bj6dbmhht.cloudfront.net/)

## Data Contract:

```
interface Info {
    interface GuessedMovie {
        title: string
        year: string
        revenue: string //rounded to the nearest $1,000,000
        director: string
        distributor: string
        genres: genre[] //see genre interface
        actors: actor[] //see actor interfaces
        tagline: string
        overview: string        
        imdbID: string
        poster: string
    }

    interface Comparison {
        correct: boolean //true if matches MysteryMovie
        collection: boolean //true if matches MysteryMovie collection
        yearComparison: int // number of years that GuessedMovie was released relative to MysteryMovie; 0 if exact, 10 is ten years after, -10 is ten years before
        revenueComparison: int // dollars that GuessedMovie made relative to MysteryMovie; 0 if exact, 1,000,000 is one million more, -1,000,000 is one million less
        directorComparison: boolean //true if director matches
        distributorComparison: boolean //true if distributor matches
        genres: genre[] //list of all shared genres
        actors: actor[] //list of all shared actors
    }
}

interface MysteryMovie {
    title: string
    year: string
    revenue: string
    director: string
    distributor: string
    genres: genre[]
    actors: actor[]
    imdbID: string
    poster: string
}

interface Hint {
    tagline: string
    overview: string
}

interface finishedGame {
    "Date": "2023/08/10",
    "Movie": "A Man Called Otto",
    "NumCorrect": 0,
    "NumIncorrect": 0,
    "Tagline": "Fall in love with the grumpiest man in America.",
    "Overview": "When a lively young family moves in next door, grumpy widower Otto Anderson meets his match in a quick-witted, pregnant woman named Marisol, leading to an unlikely friendship that turns his world upside down.",
    "Genres": [
        "Comedy",
        "Drama"
    ],
    "Actors": [
        "Tom Hanks",
        "Mariana Treviño",
        "Rachel Keller",
        "Manuel Garcia-Rulfo",
        "Cameron Britton",
        "Kailey Hyman",
        "Mike Birbiglia",
        "Elle Chapman",
        "Joe Fishel",
        "Truman Hanks"
    ],
    "Revenue": 108961677,
    "Poster": "/130H1gap9lFfiTF9iDrqNIkFvC9.jpg",
    "ReleaseYear": "2022",
    "Director": "Marc Forster",
    "Producer": "Playtone",
    "IMDB": "tt7405458",
    "Collection": ""
}


Supporting structures:

interface genre {
    name: string
}

interface actor {
    name: string
    profile_path: string
}
```

## Features:

* API Keys: With api key’s all requests are denied unless the query parameter “key” is present in the request with a valid key that is generated solely by the API and stored in the database. The API key system has 3 “levels” with each level having increasing permissions. The level system works like this. Level 1: In level 1 the user has access to the endpoints only necessary for the base game (/getMovieOptions, /getMovieDetails, /getHint, /finishGame). Level 2: In level 2 the user has access to all endpoints relating to the game (/getMovieOptions, /getMovieDetails, /getHint, /finishGame, /makeUser, /validateUser, /updateUser, /deleteUser, /getUnlimited, /getUnlimitedMovieDetails, /finishUnlimited). Level 3: In level 3 the user has access to ALL endpoints including the endpoint unique to level 3, /makeKey, which not only requires a level 3 key but also a 50 character master password that is stored in a .env file. NOTE: a level 3 key cannot be created therefore the only copy of it should be the Master Key. NOTE: when a key is created it will display the actual key value this is important to remember or write down as there is no way to find it again after creation. Please refer to the section “Endpoints” for more details about specific endpoints.



* Deployment: deployment is finished for BE and FE. The API can be found at the following link: https://lp047sxove.execute-api.us-east-1.amazonaws.com/Production/. NOTE: This link will NOT work until a valid endpoint is entered with a valid key and other needed parameters. Please refer to the section “Endpoints” for more details about specific endpoints. Frontend deployment is accessible at this link: https://d3151bj6dbmhht.cloudfront.net. To update frontend send me the code and I will update the corresponding S3 bucket and the changes will take effect in about a minute. 



* User management: allows for the creation of user which allows unlimited mode as well as tracking personal stats such as number of wins/losses for the daily mode and number of wins for unlimited mode. NOTE: a level 2 key or higher is REQUIRED in order for user management to be performed. Please refer to the section “Endpoints” for more details about specific endpoints.



* Daily movie: automatically assigns a new daily movie to be used with the /getMovieDetails and /finishGame endpoint as the first call to /getMovieDetail after midnight of the day of the last daily movie. This is the movie that the users are trying to guess. Winning this mode will increment the daily movie win counter by one as well as the user win counter if a user account is provided. Likewise, losing this mode will increment the daily movie loss counter by one as well as the user loss counter if a user account is provided. NOTE: daily mode can be played with a level 1 key. 



* Unlimited mode: NOTE: a user account must be created in order to play unlimited mode. Users can chose to randomly generate a new movie that is specific to their account with the /getUnlimited endpoint. They can use the endpoint /getUnlimitedMovieDetails to guess what they think the unlimited movie is. This process is identical to the daily movie with the big exception being that the user cannot get a hint with the /getHint endpoint like they can for the daily movie. Winning this mode will increment the users unlimited win counter by one. Losses for this mode are not tracked. NOTE: unlimited mode can only be played with a level 2 or level 3 key.  


## Endpoints:

NOTE: ALL of the following endpoints require the query parameter “key” in order to work
NOTE: Anything encompassed in quotes is a input parameter or output value (example: “User-Id”)

Level 1 Key or higher:

* /getMovieOptions (GET) - accepts 2 query parameters: “key” and “name” (name of the movie). Queries movie database to find movies with titles closest to “name” query parameter. Returns an array, “results”, of objects that contain 3 values: “title” (string), “id” (int), and “release_date” (string).
* /getMovieDetails (GET) - accepts 2 query parameters: “key” and “id” (id found from /getMovieOptions). Queries movie database to find details pertaining to the movie of the “id” query parameter. Compare guess to the set daily movie Returns the object Info highlighted in the “Data format” section.
* /finishGame (POST) - accepts a query parameter: “key”, a body containing a value: “won” (bool), and a optional header: “User-Id”. If the value “won” from the body is true, then that will be marked as a win for the daily movie as well as the user if the “User-Id” header is provided and is valid, likewise if the value “won” from the body is false, that will count as a loss for the daily movie as well as the user if the “User-Id” header is provided. Returns an instance of the finishedGame interface outlined in the data contract.
* /getHint (GET) - endpoint accepts a query parameter: “key”. Get’s overview and tagline of today’s daily movie. Returns two values: “overview” (string) and “tagline” (string).


Level 2 Key or higher:

* /makeUser (POST) - accepts a query parameter: “key”, and a body containing 2 values: “username” (string) and “password” (string). Creates a user and stores it in the database if both a “username” and “password” is found and the username does not match an already existing user, whether that user is active or inactive. Returns message containing whether the action was successful or not. 
* /validateUser (POST) - accepts a query parameter: “key”, and a body containing 2 values: “username” (string) and “password” (string). Returns “User-Id” (string) of whatever user was found with matching username and password to that entered in the body. If no matching value is found returns a message saying so. 
* /deleteUser (DELETE) - accepts a query parameter: “key”, and a header: “User-Id”. If a active user is found with that matching “User-Id” that user will be flagged as inactive and not usable by user management endpoints, however, accounts will still be unable to be created or updated if the username they entered matches that of an inactive account. Returns message containing whether the action was successful or not. 
* /updateUser (POST) - accepts a query parameter: “key”, a header: “User-Id”, and a body containing 2 values: “username” (string) and “password” (string). Updates the user matching the given “User-Id” to have a username and password set to those values from the body. The update will not work if the username matches an already existing user, whether that user is active or inactive. Returns message containing whether the action was successful or not. 
* /getUnlimited (GET) - accepts a query parameter: “key” and a header “User-Id”. Assigns a random movie to the user of matching “User-Id” with no regard to whether the movie has been repeated or not. This movie will stick with the user until /getUnlimited is called again with matching “User-Id”. Returns message containing whether the action was successful or not. 
* /getUnlimitedMovieDetails (GET) - accepts 2 query parameter: “key” and “id”, and a header “User-Id”. Queries movie database to find details pertaining to the movie of matching “id”. Unlike /getMovieDetails, this endpoint compares to the movie assigned to the user with matching “User-Id” as opposed to set daily movie. Returns the object Info highlighted in the “Data format” section.
* /finishUnlimited (POST) - accepts a query parameter: “key” and a header “User-Id”. Increments unlimited mode win counter by one for the user of matching “User-Id”. Unlike /finishGame, this endpoint does not take a boolean determining whether the game was won or not and /finishUnlimited will always assume the game was won. Returns message containing whether the action was successful or not. 


Level 3 Key or higher:

* /makeKey (POST) - accepts 4 query parameters: “key”, “manager”, “scope”, and “password”. Creates a new key which is a random 20 character string assuming the following conditions are met: a valid level 3 key has been entered, a manager has been entered detailing who is using the key, a scope between 1-2 has been entered, and the master password has been entered and is correct. The newly created key is then stored in the database for later use. Returns an object containing 5 values: “key” (string), “createdAt” (string), “numCalls” (int), “scope” (int), and “manager” (string). Please keep in mind that when a key is created you will need to write down the key as there is no way to find it again after creation. 


## Support:

Questions regarding the front-end should be directed to [Peyton Boggs](https://www.linkedin.com/in/peyton-boggs/) and questions regarding the back-end or deployment should be directed to [Jack Devitt](https://www.linkedin.com/in/jackdevitt/).

