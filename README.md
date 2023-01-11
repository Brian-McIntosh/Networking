# Networking
### An example of the OLD and NEW ways of making Network Requests

Reference: https://designcode.io/swiftui-advanced-handbook-async-await

```swift
class Network {
    
    /*
     *****************************************
     THE OLD WAY!! - with Completion Handlers
     *****************************************
    */
    func getRandomJokeWithCompletionHandler() {
        
        guard let url = URL(string: APIConstants.baseUrl) else {
            fatalError("Missing URL")
        }
        var urlRequest = URLRequest(url: url)
        urlRequest.addValue("application/json", forHTTPHeaderField: "Accept")
        
        let dataTask = URLSession.shared.dataTask(with: urlRequest) { (data, response, error) in
            
            if let error = error {
                print("Request error: ", error)
                return
            }
            
            guard (response as? HTTPURLResponse)?.statusCode == 200 else {
                return
            }
            
            guard let data = data else {
                return
            }
            
            do {
                let decodedJoke = try JSONDecoder().decode(Joke.self, from: data)
                print("OLD Completion Handler Decoded Joke", decodedJoke)
            } catch {
                print("Error decoding", error)
            }
        }
        dataTask.resume()
    }
    
    /*
     *****************************************
     THE NEW WAY!! - with Async / Await
     
     Xcode will complain that the errors THROWN within the function aren't handled, so we need to mark the getRandomFood function as THROWS, to let the program know that this function might THROW an error.
     *****************************************
    */
    func getRandomJoke() async throws {
        
        // these 3 lines are same as above:
        guard let url = URL(string: APIConstants.baseUrl) else {
            fatalError("Missing URL")
        }
        var urlRequest = URLRequest(url: url)
        urlRequest.addValue("application/json", forHTTPHeaderField: "Accept")
        
        // Since this is a network call, it can take some time to fetch the data. So we'll need to add the AWAIT keyword in front of the URLSession
        // Moreover, the URLSession call might throw an error, so we'll need to add the TRY keyword before the AWAIT.
        let (data, response) = try await URLSession.shared.data(for: urlRequest)
        
        // After getting the data and/or the response, we need to make sure that the status code of the response is 200 OK before continuing, so let's add a guard statement.
        guard (response as? HTTPURLResponse)?.statusCode == 200 else {
            fatalError("Error while fetching data")
        }
        
        // Finally, we'll decode the data that we get into the Joke model
        let decodedJoke = try JSONDecoder().decode(Joke.self, from: data)
        print("NEW Async / Await Decoded Joke", decodedJoke)
    }
}

// over in our caller, we have to handle errors:
.task { // THE NEW WAY!!
    do {
        // since this function is asynchronous and can throw an error, we'll need to add try await in front:
        let joke = try await network.getRandomFood()
    } catch {
        print("Error", error)
    }
}
```
