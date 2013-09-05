# Services

Services are each stored in their own directory. A service definition must have a config.json file and a logo.png file at minimum. The logo.png must be 228 pixels wide by 78 pixels high.

Services may have a readme.md file and/or an instructions.md file as well.

## config.json

The config section contains meta-data about the template.

* name - The unique template name. All lower-case and only the characters a-z, 0-9 and the dash.
* label - The human-readable template name, used for display.
* description - An English description of the template, used for display.

## Template

The template object holds all of the configuration details for the template. At minimum it must include the records collection, however it may include the fields collection and a hook object as well.

### Fields

A list of variable names and descriptions. These are presented to the user in the UI.

The variables are substituted into the content and name for each record when the template records are rendered into real records.

Each field may have the following attributes:

* name - When rendered the record subtitues values with the given name.
* label - Displayed to customers as the field label when the field is displayed.
* description - Displayed to customers as help text when the field is displayed.
* append - Text that is appended to the input field.
* example - Text that is subsitituted in samples that are displayed to customers.

### Records

A list of record objects which are rendered to create the real records.

Each record may have the following attributes:

* name - The host name (may be a blank string). Joined with the domain name to produce the fully-qualified record host name when rendered.
* type - The DNS type (such as "A", "CNAME", "MX", etc).
* content - The content of the record, such as an IP address or another host name. This depends on the record type.
* ttl - The time-to-live for the record.
* prio - The priority of the record. Used for MX and SRV records.

The name, type and content fields may use variables defined in the Attributes section as well as any values returned by the service hook.

Service records are rendered into the real records when the service is applied to a domain. All variables such as {{variable_name}} are replaced with attributes at this time.

In addition to custom attributes, there are also the following variables:

* domain - The domain name, such as example.com.

### Hook

Service hooks provide a way to invoke a URL when the service is applied. This can be used to execute custom behavior at services.

One example service that uses hooks is CloudFlare. The hook is used to provision the domain in the CloudFlare system.

The `url` attribute is a fully qualified URL that is invoked in one of several ways:

* POST url on create
* DELETE url on delete

The following parameters are always sent to the hook as an application/x-www-form-urlencoded body.

* domain_name: The domain name that the service is being applied to.
* user_id: The DNSimple user ID of the customer applying the service.

In addition to the parameters that are always sent, the values provided by the customer for any fields specified in the service definition are sent along as well.

For example, using the Heroku template, the body of a POST might look like the following:

    domain_name=example.com&user_id=123&app=myapp

The hook must return 200 to indicate that the operation completed successfully. Any status code other than 200 are considered an error.

The hook may return a JSON object that contains variables that is substitued in records within the service definition.

For example, the Cloudflare hook returns the following JSON:

    {
      "user_key": "some_user_key",
      "target_name": "abef4637291",
      "target_type": "CNAME"
    }

Any values returned from the hook are merged with the customer-provided values before being rendered into the services records.

## readme.md

If there is a file called readme.md in a service directory
