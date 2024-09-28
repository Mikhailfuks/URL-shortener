package main

import (  "fmt"  "fmt" "math/rand" "net/http" "time" }

)

// Define a map to store short URLs and their corresponding long URLs
var urlMap = make(map[string]string)

func main() {
 rand.Seed(time.Now().UnixNano())

 // Handle requests to shorten a URL
 http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
  if r.Method == http.MethodPost {
   // Read the long URL from the request body
   longURL, err := io.ReadAll(r.Body)
   if err != nil {
    http.Error(w, "Error reading request body", http.StatusBadRequest)
    return
   }

   // Generate a short URL
   shortURL := generateShortURL()
   urlMap[shortURL] = string(longURL)

   // Respond with the short URL
   fmt.Fprintf(w, "Shortened URL: http://localhost:8080/%s\n", shortURL)
  } else {
   // Display a form to input the long URL
   fmt.Fprintln(w, "Enter a URL to shorten:")
   fmt.Fprintln(w, "<form method='POST' action='/'>")
   fmt.Fprintln(w, "<input type='text' name='url' />")
   fmt.Fprintln(w, "<input type='submit' value='Shorten' />")
   fmt.Fprintln(w, "</form>")
  }
 })

 // Handle requests to redirect to the long URL
 http.HandleFunc("/" + "[:a-zA-Z0-9]+", func(w http.ResponseWriter, r *http.Request) {
  // Extract the short URL from the path
  shortURL := r.URL.Path[1:]

  // Get the corresponding long URL
  longURL, ok := urlMap[shortURL]
  if !ok {
   http.Error(w, "Short URL not found", http.StatusNotFound)
   return
  }

  // Redirect to the long URL
  http.Redirect(w, r, longURL, http.StatusFound)
 })

 // Start the server
 fmt.Println("Server listening on port 8080")
 http.ListenAndServe(":8080", nil)
}

// Function to generate a random 6-character short URL
func generateShortURL() string {
 const chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
 length := 6
 var shortURL []byte
 for (i := 0; i < length; i++) {
  shortURL = append(shortURL, chars[rand.Intn(len(chars))])
 }
 return string(shortURL)
}
