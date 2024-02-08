---
layout: post
title:  "Remember Me functionality"
date:   2020-03-24
---

From the Rails Tutorial and Railscast

Overview of the steps to implement this:

1. Create a random string of digits for use as a remember token.
2. Place the token in the browser cookies with an expiration date far in the future.
3. Save the hash digest of the token to the database.
4. Place an encrypted version of the user’s id in the browser cookies.
5. When presented with a cookie containing a persistent user id, find the user in the database using the given id and verify that the remember token cookie matches the associated hash digest from the database.

