# A small collection of Bambda filters for Burp Suite

## Open-redirects
Checks any response with a 3xx response code, if the request contains parameters and they are of type URL and start with (http|https|//) the filter will check if the response's location header matches the parameter value and return true if it does
```java
if (!requestResponse.hasResponse()) {
  return false;
}
var response = requestResponse.response();
if (response.isStatusCodeClass(StatusCodeClass.CLASS_3XX_REDIRECTION)) {
  var parameters = requestResponse.request().parameters();
  for (var param : parameters) {
    if (param.type() != HttpParameterType.URL) {
      return false;
    }
    var decodedParam = utilities().urlUtils().decode(param.value()).toLowerCase();
    if (decodedParam.startsWith("http") || decodedParam.startsWith("https") || decodedParam.startsWith("//")) {
      var LocationValue = requestResponse.response().headerValue("Location").toLowerCase();
      if (LocationValue.startsWith(decodedParam)) {
        return true;
      }
    }
  }
}
return false;
```
## Cachable responses
Checks the headers of every response, if the header name contains the string "cache" and the header value contains "hit" or "miss" it will return true
```java
if (!requestResponse.hasResponse()) {
  return false;
}
var headers = requestResponse.response().headers();
for (var header : headers) {
  if (header.name().toLowerCase().contains("cache") && (header.value().toLowerCase().contains("hit") || header.value().toLowerCase().contains("miss"))) {
    return true;
  }
}
return false;
```
## Create a wordlist of unique parameters
Checks every request for parameters of type URL, if any are found and they are unique they will be saved to the path specified in the file variable, the generated file can later be used as a custom wordlist in an extension such as Param Miner
```java
var request = requestResponse.request();
// Parameter Type can be modified to your liking(URL,BODY,JSON,COOKIE,XML)
if (!request.hasParameters(HttpParameterType.URL)) {
    return false;
}

var parameters = request.parameters();
var uniqueParameters = new HashSet<String>();
var file = new File("/path/to/output.txt");
if (!file.exists()) {
    file.createNewFile();
}

var reader = new BufferedReader(new FileReader(file));
var writer = new BufferedWriter(new FileWriter(file, true));
while (reader.ready()) {
    uniqueParameters.add(reader.readLine());
}
reader.close();
for (var param : parameters) {
    // Parameter Type can be modified to your liking(URL,BODY,JSON,COOKIE,XML)
    if (param.type() == HttpParameterType.URL && !uniqueParameters.contains(param.name())) {
        writer.write(param.name());
        writer.newLine();
    }
}
writer.close();
return true;
```
[Twitter - @_0x999](https://twitter.com/_0x999)
