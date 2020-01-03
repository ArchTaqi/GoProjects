package main

import (
	"net/http"

	"html/template"

	"github.com/go-redis/redis"
	"github.com/gorilla/mux"
)

var client *redis.Client
var templates *template.Template

func main() {
	client = redis.NewClient(&redis.Options{Addr: "localhost:6379"})
	templates = template.Must(template.ParseGlob("templates/*.html"))
	r := mux.NewRouter()
	r.HandleFunc("/", indexHandler).Methods("GET")
	r.HandleFunc("/", indexPostHandler).Methods("POST")

	r.HandleFunc("/tos", authRequired(indexHandler)).Methods("GET")

	fs := http.FileServer(http.Dir("./static/"))
	r.PathPrefix("/static/").Handler(http.StripPrefix("/static/", fs))
	http.Handle("/", r)

	http.ListenAndServe(":8080", r)
}

func authRequired(handler http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		login := false
		if !login {
			http.Redirect(w, r, "/", 302)
		}
		handler.ServeHTTP(w, r)
	}
}

func indexHandler(w http.ResponseWriter, r *http.Request) {
	comments, err := client.LRange("comments", 0, 10).Result()
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		w.Write([]byte("Internal server error"))
		return
	}
	templates.ExecuteTemplate(w, "index.html", comments)
}

func indexPostHandler(w http.ResponseWriter, r *http.Request) {
	r.ParseForm()
	comment := r.PostForm.Get("comment")
	err := client.LPush("comments", comment).Err()
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		w.Write([]byte("Internal server error"))
		return
	}
	http.Redirect(w, r, "/", 302)
}
