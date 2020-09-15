<h2>Yet Another Yet Another Cat Challenge</h2>

<h3>Description<h3>

<blockquote>Yet Another Cat Challenge for you! Just another boring add/remove/share note app, but with 8 different cats!

http://yayacc.zajebistyc.tf</blockquote>



<p>One day when I was chatting with my friend <a href="https://gynvael.coldwind.pl/">Gynvael</a>, he told me about one hard XSS challenge from CONFidence CTF 2020 that he faced. So I decided to give it a try and see what is going on, two hints that he gave me were : 1.HTML injection 2.Theme selection where XSS will happen. <br>
So in this challenge after we login, we get to the page where we can add "note". On view page source we can see this:
</p>

![addnote](https://github.com/DejanJS/CONFidence-CTF-2020-yayacc/blob/master/entrypage.png)

```javascript
<html>
<head>
    <title>Yet Another Cat Challenge</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css"
          integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">
    <script src="/flag?var=flag" nonce="0vOlzlwTCvS5btb6IkPB1Q"></script>
    <script src="https://www.google.com/recaptcha/api.js?render=6Le09cUZAAAAAN4r7Ushpf0tdpAowD0LNE1gNsHI"></script>
    <script nonce="0vOlzlwTCvS5btb6IkPB1Q">
        window.onload = function()
        {
            document.getElementById('hello').innerText += ' ' + flag;
        }
        grecaptcha.ready(function() {
            grecaptcha.execute('6Le09cUZAAAAAN4r7Ushpf0tdpAowD0LNE1gNsHI', {action:'validate_captcha'})
                  .then(function(token) {
            // add token value to form
            captchas = document.getElementsByName('g-recaptcha-response')
            for(var i = 0; i < captchas.length; i++)
                captchas[i].value = token;
        });
    });
    
    </script>
</head>
<body>
<div id="hello">
    Hello pkellerman, the flag is:
</div>
<ul class="list-group">
    
</ul>
<a href="/note" class="btn btn-primary">Add note</a>
</body>
</html>
```

<p>We can see there is script tag with src to <code>/flag?var=flag</code> and if we go to that path we won't get the flag obviously because we are not an admin</p>