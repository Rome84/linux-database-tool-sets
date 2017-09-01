# Print webpage into pdf or image

## Install wkhtmltopdf
```
 $ wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz

 $ tar xf wkhtmltox-0.12.4_linux-generic-amd64.tar.xz

 # example 
 $ wkhtmltopdf http://google.com google.pdf

```

## Install pdftk
 ``wkhtmltopdf`` is printing multiple pages PDFS. In some case, only the first page is needed. ``pdftk`` is used to output only the first page. 

 ```
 sudo yum localinstall https://www.linuxglobal.com/static/blog/pdftk-2.02-1.el7.x86_64.rpm
 ```

 ### Print only the first page of a webpage
 ```
 $ wkhtmltopdf http://stackoverflow.com temp.pdf
 $ pdftk temp.pdf cat 1 output output.pdf
 $ rm temp.pdf
 ```

 ### Convert pdf to png 
  ```
 $ convert output.pdf output.png
 ```
