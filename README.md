# Build-your-own-Movie-Recommendation-App-in-SwiftUI
Building a Movie Recommendation App in SwiftUI involves several key steps, such as displaying a list of movies, recommending movies based on user preferences, and interacting with an API for movie data. For simplicity, I will walk you through creating a basic app that pulls movie data from an API (such as The Movie Database API), displays a list of movies, and allows the user to recommend movies.
Steps:

    Create a new SwiftUI project in Xcode.
    Use the Movie Database API to get movie data (you'll need an API key).
    Display movies in a list with movie details.
    Provide movie recommendations based on user preferences.

Step 1: Create a New SwiftUI Project

    Open Xcode and create a new SwiftUI App project.
    Name your project (e.g., MovieRecommendationApp).
    Choose SwiftUI as the interface and Swift as the language.

Step 2: Get API Key from The Movie Database (TMDb)

    Go to TMDb and create an account if you don't have one.
    Visit the API section to get your API key.
    Use this API key in your app to make requests.

Step 3: Create Model for Movies

In MovieRecommendationApp, create a Movie.swift file to define the movie structure:

import Foundation

// Movie model that conforms to Decodable to handle API response
struct Movie: Identifiable, Decodable {
    let id: Int
    let title: String
    let overview: String
    let posterPath: String
    let releaseDate: String
    
    var posterURL: URL? {
        return URL(string: "https://image.tmdb.org/t/p/w500\(posterPath)")
    }
}

// Structure to decode the response from the API
struct MovieResponse: Decodable {
    let results: [Movie]
}

Step 4: Create a Network Manager

Now create a NetworkManager.swift file to fetch movie data from the API:

import Foundation

class NetworkManager: ObservableObject {
    @Published var movies = [Movie]()
    
    private let apiKey = "YOUR_API_KEY" // Replace with your TMDb API key
    private let baseURL = "https://api.themoviedb.org/3"
    
    func fetchMovies() {
        guard let url = URL(string: "\(baseURL)/discover/movie?api_key=\(apiKey)&language=en-US&sort_by=popularity.desc&page=1") else {
            return
        }
        
        let task = URLSession.shared.dataTask(with: url) { data, response, error in
            if let data = data {
                let decoder = JSONDecoder()
                do {
                    let movieResponse = try decoder.decode(MovieResponse.self, from: data)
                    DispatchQueue.main.async {
                        self.movies = movieResponse.results
                    }
                } catch {
                    print("Error decoding movie data: \(error)")
                }
            }
        }
        
        task.resume()
    }
}

Step 5: Create the Movie List View

Now, let's create a view to display the list of movies.

import SwiftUI

struct MovieListView: View {
    @StateObject private var networkManager = NetworkManager()
    
    var body: some View {
        NavigationView {
            List(networkManager.movies) { movie in
                NavigationLink(destination: MovieDetailView(movie: movie)) {
                    HStack {
                        if let posterURL = movie.posterURL {
                            AsyncImage(url: posterURL) { image in
                                image.resizable().scaledToFit()
                            } placeholder: {
                                Color.gray
                            }
                            .frame(width: 60, height: 90)
                        }
                        VStack(alignment: .leading) {
                            Text(movie.title)
                                .font(.headline)
                            Text(movie.releaseDate)
                                .font(.subheadline)
                                .foregroundColor(.gray)
                        }
                    }
                }
            }
            .navigationTitle("Movies")
            .onAppear {
                networkManager.fetchMovies()
            }
        }
    }
}

Step 6: Create the Movie Detail View

This view will show the details of each movie when tapped.

import SwiftUI

struct MovieDetailView: View {
    let movie: Movie
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading) {
                if let posterURL = movie.posterURL {
                    AsyncImage(url: posterURL) { image in
                        image.resizable().scaledToFit()
                    } placeholder: {
                        Color.gray
                    }
                    .frame(maxWidth: .infinity)
                    .cornerRadius(10)
                }
                
                Text(movie.title)
                    .font(.title)
                    .bold()
                    .padding(.top)
                
                Text(movie.releaseDate)
                    .font(.subheadline)
                    .foregroundColor(.gray)
                
                Text(movie.overview)
                    .padding(.top)
            }
            .padding()
        }
        .navigationTitle(movie.title)
    }
}

Step 7: Display the Movie List in the Main Content View

Finally, let's display the movie list view in the main ContentView.swift:

import SwiftUI

struct ContentView: View {
    var body: some View {
        MovieListView()
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

Step 8: Run Your App

    Build and run your app.
    You should see a list of popular movies fetched from the TMDb API. When you tap on a movie, it will show the details such as the movie's title, poster, release date, and overview.

Conclusion

In this app, we created a basic Movie Recommendation App in SwiftUI that:

    Fetches movie data from The Movie Database API.
    Displays a list of movies using List in SwiftUI.
    Displays detailed information about each movie in a separate view when clicked.

You can extend this app by adding additional features like:

    Movie recommendations based on user ratings or preferences.
    Search functionality to find movies.
    Integration with a backend to store user preferences and provide personalized recommendations.
