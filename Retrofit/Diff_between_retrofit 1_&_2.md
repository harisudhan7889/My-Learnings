# Difference between Retrofit 1 and Retrofit 2 

### Retrofit 1:
- Separate Synchrnous and Asynchrous call declarations
  Example: 
  ```
  public interface APIService {
    /*Synhcronous call*/
    @POST("/list")
    Repo loadRepo();
    
    /*Asynchrounous call*/
    @POST("/list")
    void loadRepo(Callback<Repo> cb);
    }
    ```
- GsonConverter is included in the package and is automatically initiated upon RestAdapter creation. As a result, the json result from server would be automatically parsed to the defined Data Access Object (DAO).
- OkHttp is set to optional in Retrofit 1.9. If you want to let Retrofit use OkHttp as HTTP connection interface, you have to manually include okhttp as a dependency yourself.
- If the fetched response couldn't be parsed into the defined Object, failure will be called.

### Retrofit 2:
- No need of separate Synchronous and Asynchronous service declaration.
  Example:
  ```
    public interface APIService {
    @POST("/list")
    Call<Repo> loadRepo();
    }
  ```
- Converter is not included in the package anymore. You need to plug a Converter in yourself or Retrofit will be able to accept only the String result. As a result, Retrofit 2.0 doesn't depend on Gson anymore.
- OkHttp is now required and is automatically set as a dependency.
- Whether the response is be able to parse or not, onResponse will be always called. In the case the result couldn't be parsed into the Object, response.body() will return as null. If there is any problem on the response, for example, 404 Not Found. onResponse will also be called. You can retrieve the error body from response.errorBody().string().
- Beside declaring interface with Call<T> pattern, we also could declare our own type as well, for example, MyCall<T>. The mechanic is called "CallAdapter" which is available on Retrofit 2.0