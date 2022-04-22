# WordPress Plugin - Elementor 3.6.0 3.6.1 3.6.2 Remote Code Execution

```
Google Dork: none
Date: April 16 2022
Exploit Author: AkuCyberSec (https://github.com/AkuCyberSec)
Vendor Homepage: https://elementor.com/
Software Link: https://wordpress.org/plugins/elementor/advanced/ (scroll down to select the version)
Version: 3.6.0, 3.6.1, 3.62
Tested on: WordPress 5.9.3 (os-independent since this exploit does NOT provide the payload)
CVE : CVE-2022-1329
```

## VULNERABILITY DESCRIPTION

The WordPress plugin called Elementor (v. 3.6.0, 3.6.1, 3.6.2) has a vulnerability that allows any authenticated user to upload and execute any PHP file.

This vulnerability, in the OWASP TOP 10 2021, is placed in position #1 (Broken Access Control)

The file that contains this vulnerability is elementor/core/app/modules/onboarding/module.php

At the end of this file you can find this code:
```
add_action( 'admin_init', function() {
		if ( wp_doing_ajax() &&
			isset( $_POST['action'] ) &&
			isset( $_POST['_nonce'] ) &&
			wp_verify_nonce( $_POST['_nonce'], Ajax::NONCE_KEY )
		) {
			$this->maybe_handle_ajax();
		}
	} );
```

This code is triggered whenever ANY user account visits /wp-admin

In order to work we need the following 4 things:

1. The call must be an "ajax call" (wp_doing_ajax()) and the method must be POST. In order to do this, we only need to call /wp-admin/admin-ajax.php
2. The parameter "action" must be "elementor_upload_and_install_pro" (check out the function named maybe_handle_ajax() in the same file)
3. The parameter "_nonce" must be retrieved after login by inspecting the /wp-admin page (this exploit does this in DoLogin function)
4. The parameter "fileToUpload" must contain the ZIP archive we want to upload (check out the function named upload_and_install_pro() in the same file)

The file we upload must have the following structure:

1. It must be a ZIP file. You can name it as you want.
2. It must contain a folder called "elementor-pro"
3. This folder must contain a file named "elementor-pro.php"

This file will be YOUR payload (e.g. PHP Reverse Shell or anything else)

WARNING: The fake plugin we upload will be activated by Elementor, this means that each time we visit any page we trigger our payload.

If it tries, for example, to connect to an offline host, it could lead to a Denial of Service.

In order to prevent this, I suggest you to use some variable to activate the payload.

Something like this (visit anypage.php?activate=1 in order to continue with the actual payload):

```
if (!isset($_GET['activate']))
return;
```
