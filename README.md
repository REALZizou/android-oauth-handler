# CodePath OAuth Handler

This library is an Android library for managing OAuth requests with an extremely easy
approach that keeps the details of the OAuth process abstracted from the end-user developer.

This library leverages a few key libraries underneath to power the functionality:

 * [scribe-java](https://github.com/fernandezpablo85/scribe-java) - Simple OAuth library for handling the authentication flow.
 * [Android Async HTTP](https://github.com/loopj/android-async-http) - Simple asynchronous HTTP requests with JSON parsing.

## Installation

Just download the latest [jar file](https://www.dropbox.com/s/mryygauuao58mqp/codepath-oauth-0.2.0.jar) for this library and drop the jar into your "libs" folder in your Android project.

## Getting Started

This library is very simple to use and simply requires you to create an Activity that is used for authenticating with OAuth and ultimately give your application access to an authenticated API.

### Creating a REST Client

The first step is to create a REST Client that will be used to access the authenticated APIs
within your application. A REST Client is defined in the structure below:

```java
public class TwitterClient extends OAuthBaseClient {
    public static final Class<? extends Api> REST_API_CLASS = TwitterApi.class;
    public static final String REST_URL = "http://api.twitter.com";
    public static final String REST_CONSUMER_KEY = "SOME_KEY_HERE";
    public static final String REST_CONSUMER_SECRET = "SOME_SECRET_HERE";
    public static final String REST_CALLBACK_URL = "oauth://arbitraryname";

    public TwitterClient(Context context) {
        super(context, REST_API_CLASS, REST_URL,
          REST_CONSUMER_KEY, REST_CONSUMER_SECRET, REST_CALLBACK_URL);
    }

    // ENDPOINTS BELOW

    public void getHomeTimeline(int page, AsyncHttpResponseHandler handler) {
      String apiUrl = getApiUrl("statuses/home_timeline.json");
      RequestParams params = new RequestParams();
      params.put("page", String.valueOf(page));
      client.get(apiUrl, params, handler);
    }
}
```

Configure the `REST_API_CLASS`, `REST_URL`, `REST_CONSUMER_KEY`, `REST_CONSUMER_SECRET` based on the values needed to connect to your particular API. The `REST_URL` should be the base URL used for connecting to the API (i.e `https://api.twitter.com`). The `REST_API_CLASS` should be the class defining the [service](https://github.com/fernandezpablo85/scribe-java/tree/master/src/main/java/org/scribe/builder/api) you wish to connect to. Check out the [full list of services](https://github.com/fernandezpablo85/scribe-java/tree/master/src/main/java/org/scribe/builder/api) you can select (i.e `FlickrApi.class`).

### Creating a LoginActivity

The next step is to create a `LoginActivity` to use for logging into an OAuth service:

```java
public class LoginActivity extends OAuthLoginActivity<FlickrClient> {
  // This fires once the user is authenticated, or fires immediately
  // if the user is already authenticated.
  @Override
  public void onLoginSuccess() {
    Intent i = new Intent(this, PhotosActivity.class);
    startActivity(i);
  }

  // Fires if the authentication process fails for any reason.
  @Override
  public void onLoginFailure(Exception e) {
      e.printStackTrace();
  }

  // Method to be called to begin the authentication process
  // assuming user is not authenticated.
  // Typically used as an event listener for a button for the user to press.
  public void loginToRest(View view) {
      getClient().connect();
  }
}
```

A few notes for your `LoginActivity`:

 * Your activity must extend from `OAuthLoginActivity<SomeRestClient>`
 * Your activity must implement `onLoginSuccess` and `onLoginFailure`
 * The `onLoginSuccess` should launch an "authenticated" activity.
 * The activity should have a button or other view a user can press to trigger authentication.

### Using the REST Client

These endpoint methods will automatically execute asynchronous requests signed with the authenticated access token anywhere your application. To use JSON endpoints, simply invoke the method
with a `JsonHttpResponseHandler` handler:

```java
// SomeActivity.java
RestClient client = RestClientApp.getRestClient();
client.getHomeTimeline(1, new JsonHttpResponseHandler() {
  public void onSuccess(JSONArray json) {
    // Response is automatically parsed into a JSONArray
    // json.getJSONObject(0).getLong("id");
  }
});
```

Based on the JSON response (array or object), you need to declare the expected type inside the `onSuccess` signature i.e `public void onSuccess(JSONObject json)`. If the endpoint does not return JSON, then you can use the `AsyncHttpResponseHandler`:

```java
RestClient client = RestClientApp.getRestClient();
client.get("http://www.google.com", new AsyncHttpResponseHandler() {
    @Override
    public void onSuccess(String response) {
        System.out.println(response);
    }
});
```

Check out [Android Async HTTP Docs](http://loopj.com/android-async-http/) for more request creation details.