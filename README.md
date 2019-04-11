# SAMLAuthenticator for JupyterHub

This is a SAML Authenticator for JupyterHub. With this code (and a little elbow grease), you can integrate your JupyterHub instance with a previously setup SAML Single Sign-on system!

## Set Up

This set up section assumes that python 3.6+, pip, and JupyterHub are already set up on the target machine.

If the `jupyterhub_config.py` file has not been generated, this would be a good time to generate it. For a primer on generating the config file, read [here](https://jupyterhub.readthedocs.io/en/stable/getting-started/config-basics.html).

### Installation

In the context in which JupyterHub will be run, install the SAML Authenticator.

```sh
pip install jupyterhub-samlauthenticator
```

### Configuration

Open the `jupyterhub_config.py` file in an available text editor.

Change the configured value of the `authenticator_class` to be `samlauthenticator.SAMLAuthenticator`.

Configure one of the accepted metadata sources. The SAMLAuthenticator can get metadata from three sources:
1. The most preferable option is to configure the SAMLAuthenticator to use a metadata file. This can be done by setting the `metadata_filepath` field of the `SAMLAuthenticator` class to the *_fully justified filepath_* of the metadata file.
1. Another option is to dump the full metadata xml into the JupyterHub configuration file. This is not great because it clutters up the configuration file with a lot of extraneous data. This can be done by setting the `metadata_content` field of the SAMLAuthenticator class.
1. Finally, the least preferable option of the three is to get the metadata from a web request each time a user attempts to log into the server. This is _not recommended_ because DNS poisoning attacks could let a malicious actor impersonate the IdP and gain access to any user private files on the server. However, if this is the configuration that is required, set the `metadata_url` field and the metadata will be refreshed every time a user attempts to log in to the JupyterHub server.

This is all the configuration the Authenticator _usually_ requires, but there are more configuration options to go through.

If the user that should be created and logged in from a given SAML Response is _not_ specified by the NameID element in the SAML Assertion, an alternate field can be specified. Replace the `xpath_username_location` field in the `SAMLAuthenticator` with an XPath that points to the desired field in the SAML Assertion. Note that this value must be able to be compiled to an XPath by Python's `lxml` module. The namespaces that will be present for this XPath are as follows:

```py
{
    'ds'   : 'http://www.w3.org/2000/09/xmldsig#',
    'saml' : 'urn:oasis:names:tc:SAML:2.0:assertion',
    'samlp': 'urn:oasis:names:tc:SAML:2.0:protocol'
}
```

The SAMLAuthenticator expects the SAML Response to be in the `SAMLResponse` field of the POST request that the user makes to authenticate themselves. If this expectation does not hold for a given environment, then the `login_post_field` property of the SAMLAuthenticator should be set to the correct field.

A SAML Audience and Recipient can be defined on the IdP to prevent a malicious service from using a SAML Response to inappropriately authenticate to non-malicious services. If either of these values is set by the IdP, they can be checked by setting the `audience` and `recipient` fields on the SAMLAuthenticator.

By default, the SAMLAuthenticator expects the `NotOnOrAfter` and `NotBefore` fields to be of the format `{four-digit-year}-{two-digit-month}-{two-digit-day}T{two-digit-24-hour-hour-value}:{two-digit-minute}:{two-digit-second}Z` where T and Z are character literals. If this is not a good assumption, an alternate time string can be provided by setting the `time_format_string` value of the SAMLAuthenticator. This string will be consumed by Python's [`datetime.strptime()`](https://docs.python.org/3.6/library/datetime.html#datetime.datetime.strptime), so it might be helpful to read up on [the `strftime()` and `strptime()` behavior](https://docs.python.org/3.6/library/datetime.html#strftime-strptime-behavior).

If the timezone being passed in by the `NotOnOrAfter` and `NotBefore` fields cannot be read by `strptime()`, don't fear! So long as the timezone that the IdP resides in is known, it's possible to set the IdP's timezone. Set the `idp_timezone` field to a string that uniquely designates a timezone that can be looked up by [`pytz`](https://pypi.org/project/pytz/), and login should be able to continue.

#### Example Configurations

```py
# A simple example configuration.
## Class for authenticating users.
c.JupyterHub.authenticator_class = 'samlauthenticator.SAMLAuthenticator'

# Where the SAML IdP's metadata is stored.
c.SAMLAuthenticator.metadata_filepath = '/etc/jupyterhub/metadata.xml'
```

```py
# A complex example configuration.
## Class for authenticating users.
c.JupyterHub.authenticator_class = 'samlauthenticator.SAMLAuthenticator'

# Where the SAML IdP's metadata is stored.
c.SAMLAuthenticator.metadata_filepath = '/etc/jupyterhub/metadata.xml'

# A field was placed in the SAML Response that contains the user's first name and last name separated by a period.
# Let's use that for the username.
c.SAMLAuthenticator.xpath_username_location = '//saml:Attribute[@Name="DottedName"]/saml:AttributeValue/text()'

# The IdP is sending the SAML Response in a field named 'R'
c.SAMLAuthenticator.login_post_field = 'R'

# We want to make sure that we're the only one receiving this SAML Response
c.SAMLAuthenticator.audience = 'jupyterhub.myorg.com'
c.SAMLAuthenticator.recipient = 'https://jupyterhub.myorg.com/hub/login'

# The IdP is sending dates in the form 'Tue July 20, 2020 18:30:21'
c.SAMLAuthenticator.time_format_string = '%a %B %d, %Y %H:%M%S'

# Looks like we can't get the timezone from the previous string - we need to set it
c.SAMLAuthenticator.idp_timezone = 'US/Eastern'
```

## Developing and Contributing

TODO






