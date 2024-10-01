See [[ipython]] for other usage
### Making an http request with python

`#!/usr/bin/env python3`
`import requests`

`url = "http://127.0.0.1:80"`

`print(requests.get(url).text)`

#### Set headers
`requests,get(url, headers={"Host":"Value"})`


### encoded url
`import requests 
`url = "http://127.0.0.1:80/358777ed%201b73bc0f/4f1a7d43%20cb50ed5a" response = requests.get(url) 
`print(response.text)`