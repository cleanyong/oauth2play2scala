oauth2play2scala
================

oauth2 auth and token server for Play 2 Scala API. Ported from Apache Amber.


As Play2 document mentioned that impl an oauth2 server just piece of cake, so I give it a try, by found out not so easy.Because oauth2 spec if very board.

Then I try to find the oauth2 impl for Play2.

1. securesocial is not an oauth2 server
2. deadbolt is not an oauth2 server
3. scalatra/oauth2-server is not a Play2 plugin
4. raibledesigns.com/rd/entry/secure_json_services_with_play is not an oauth2 server

OK, let me try using Apache Amber from Scala/Play2, this one works with changed HttpServletRequest to play.api.mvc.Request !

So, everyone wants an oauth2 impl in Play2 and save you 9 hours! ( I used more then this :-)  )


Here is the usage:
1. Clone this repo and build the jar with mvn jar:jar
2. Copy the jar to Play2 project under lib/  
3. put this in your routes:
GET    /oauth2/auth    			controllers.Application.auth()
POST 	/oauth2/token    			controllers.Application.token()

4. create the action like this:
  def auth = Action { implicit request =>
    try {
      //dynamically recognize an OAuth profile based on request characteristic (params,
      // method, content type etc.), perform validation
      val oauthRequest = new OAuthAuthzRequest(request)

      //some code ....
      if (oauthRequest.getClientSecret() == null) {
        //              throw OAuthProblemException.error("404", "no such user")
      }

      //build OAuth response
      val resp = OAuthASResponse.
        authorizationResponse(request, 302).
        setCode("hfsfhkjsdf").
        location("http://app-host:9000/authz").
        buildQueryMessage();
      Found(resp.getLocationUri())

      //if something goes wrong
    } catch {
      case ex: OAuthProblemException =>

        try {
          val resp = OAuthResponse.
            errorResponse(404).error(ex).location("http://app-host:9000/erro").buildQueryMessage();
          Redirect(resp.getLocationUri());

        } catch {
          case e: OAuthSystemException =>
            e.printStackTrace();
            InternalServerError(e.getMessage());
        }
      case ex: OAuthSystemException =>
        ex.printStackTrace()
        InternalServerError(ex.getMessage())

    }

  }

  def token = Action { implicit request =>

    val oauthIssuerImpl: OAuthIssuer = new OAuthIssuerImpl(new MD5Generator());

    try {
      val oauthRequest: OAuthTokenRequest = new OAuthTokenRequest(request);

      val authzCode = oauthRequest.getCode();

      // some code
      // System.out.println(authzCode);

      val accessToken = oauthIssuerImpl.accessToken();
      val refreshToken = oauthIssuerImpl.refreshToken();

      // some code
      System.out.println(accessToken);
      System.out.println(refreshToken);

      val r = OAuthASResponse
        .tokenResponse(200) //HttpServletResponse.SC_OK
        .setAccessToken(accessToken)
        .setExpiresIn("3600")
        .setRefreshToken(refreshToken)
        .buildJSONMessage();

      Ok(r.getBody());

      //if something goes wrong
    } catch {
      case ex: OAuthProblemException =>
        var r: OAuthResponse = null;
        try {
          r = OAuthResponse
            .errorResponse(401)
            .error(ex)
            .buildJSONMessage();
        } catch {
          case e: OAuthSystemException =>
            e.printStackTrace();
            InternalServerError(e.getMessage());
        }

        InternalServerError(r.getBody());

      case ex: OAuthSystemException =>
        ex.printStackTrace()
        InternalServerError(ex.getMessage())

    }

  }
  
  
6. Please figure out create your KDC and userRealm yourself. Have fun !
