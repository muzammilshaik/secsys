---
# the default layout is 'page'
icon: fab fa-github
order: 5
layout: none
#redirect_to: https://github.com/s3csys
---

<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Redirecting...</title>
  </head>
  <body>
    <script>
      const opened = window.open("https://github.com/s3csys", "_blank");
      if (opened) {
        // Redirect the current tab to your homepage or any other page
        window.location.href = "/";
      } else {
        // Fallback if the popup was blocked
        document.write('Click <a href="https://github.com/s3csys" target="_blank">here</a> to open the GitHub repo.');
      }
    </script>
  </body>
</html>
