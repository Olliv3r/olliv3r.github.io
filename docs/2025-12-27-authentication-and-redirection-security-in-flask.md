layout: page
title: "Autenticação e redirecionamento seguro no flask"
permalink: olliv3r.github.io/authentication-and-redirection-security-in-flask.html
<!-- date: 2025-12-27 05:24:34 -0000 -->
categories: DesenvolvimentoWeb seguranca backend frontend fullstack
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Autenticação e Redirecionamento Seguro no Flask</title>
  <style>
    /* Reset básico */
    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background-color: #1e1e2f;
      color: #e0e0e0;
      line-height: 1.6;
      padding-top: 60px; /* espaço para menu fixo */
    }

    a { color: #f39c12; text-decoration: none; }
    a:hover { text-decoration: underline; }

    header {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      background-color: #2c2c3c;
      padding: 15px 20px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.5);
      z-index: 1000;
    }

    header nav a {
      margin-right: 20px;
      font-weight: bold;
    }

    main {
      max-width: 900px;
      margin: 0 auto;
      padding: 20px;
    }

    footer {
      background-color: #2c2c3c;
      color: #bbb;
      text-align: center;
      padding: 15px 20px;
      margin-top: 40px;
      font-size: 0.9em;
    }

    h1, h2, h3 {
      color: #f39c12;
      margin-top: 1.5em;
    }

    h1 { font-size: 2em; }
    h2 { font-size: 1.6em; }
    h3 { font-size: 1.3em; }

    code {
      background-color: #2c2c3c;
      color: #f1c40f;
      padding: 2px 4px;
      border-radius: 3px;
      font-family: 'Courier New', monospace;
    }

    pre {
      background-color: #2c2c3c;
      padding: 15px;
      border-radius: 6px;
      overflow-x: auto;
      color: #f1c40f;
      margin: 15px 0;
    }

    ul { margin-left: 20px; margin-bottom: 15px; }

    table {
      border-collapse: collapse;
      width: 100%;
      margin: 20px 0;
      color: #e0e0e0;
    }

    th, td {
      border: 1px solid #444;
      padding: 10px;
      text-align: left;
    }

    th { background-color: #34495e; color: #f39c12; }
    tr:nth-child(even) { background-color: #2c2c3c; }

    hr { border: 0; border-top: 1px solid #444; margin: 30px 0; }

    @media (max-width: 600px) {
      body { padding-top: 80px; }
      header nav a { display: block; margin-bottom: 10px; }
      pre { font-size: 0.9em; }
    }
  </style>
</head>
<body>

<header>
  <nav>
    <a href="#next">Next</a>
    <a href="#login">Login</a>
    <a href="#session">Sessão</a>
    <a href="#ajax">AJAX</a>
    <a href="#urls">URLs</a>
    <a href="#checklist">Checklist</a>
  </nav>
</header>

<main>

<h1>Autenticação e Redirecionamento Seguro no Flask</h1>

<p>Este artigo apresenta boas práticas para lidar com <code>next</code>, AJAX e login seguro no Flask, garantindo que o usuário seja redirecionado corretamente para a página desejada após autenticação.</p>

<hr>

<h2 id="next">1. Entendendo o <code>next</code></h2>
<ul>
  <li><code>next</code> representa a página que o usuário tentou acessar antes de ser redirecionado para login.</li>
  <li>Problema comum: query string só funciona no GET, e desaparece no POST.</li>
</ul>
<p>Dica: transporte <code>next</code> via hidden field no formulário ou via sessão.</p>

<hr>

<h2 id="login">2. Login com WTForms + <code>next</code></h2>

<h3>Formulário WTForms</h3>
<pre><code>class SigninForm(FlaskForm):
    email = StringField("Email")
    password = PasswordField("Senha")
    remember_me = BooleanField("Lembrar-me")
    next = HiddenField()
    submit = SubmitField("Entrar")
</code></pre>

<h3>Template HTML</h3>
<pre><code>&lt;form method="POST"&gt;
  {{ form.hidden_tag() }}
  {{ form.next(value=next) }}
  {{ form.email }}
  {{ form.password }}
  {{ form.remember_me }}
  {{ form.submit }}
&lt;/form&gt;
</code></pre>

<h3>Rota de login</h3>
<pre><code># GET
next_page = request.args.get("next")
return render_template("auth/signin.html", form=form, next=next_page)

# POST
next_page = form.next.data
if next_page and is_safe_url(next_page):
    return redirect(next_page)
</code></pre>

<hr>

<h2 id="session">3. Versão robusta: <code>next</code> na sessão</h2>
<pre><code>@login_manager.unauthorized_handler
def unauthorized():
    if request.headers.get("X-Requested-With") == "XMLHttpRequest":
        return "", 401
    session["next"] = request.path
    return redirect(url_for("auth.signin"))
</code></pre>

<p>Após o login:</p>
<pre><code>next_page = session.pop("next", None)
if next_page and is_safe_url(next_page):
    return redirect(next_page)
return redirect(url_for("main.index"))
</code></pre>

<ul>
  <li>URL limpa (sem <code>?next=</code>)</li>
  <li>Fluxo seguro</li>
  <li>Imune à manipulação pelo usuário</li>
</ul>

<hr>

<h2 id="ajax">4. AJAX e autenticação</h2>
<ul>
  <li>Rotas de polling não devem alterar <code>next</code>.</li>
  <li>Não usar <code>@login_required</code> em AJAX; apenas verificar <code>current_user.is_authenticated</code>.</li>
</ul>

<pre><code>if not current_user.is_authenticated:
    return "", 401
</code></pre>

<p>Exemplo seguro:</p>
<pre><code>$.ajax({
  url: "/notifications/render",
  type: "GET",
  success: function(data) {
    $("#notifications_container").html(data);
  }
});
</code></pre>

<hr>

<h2 id="urls">5. URLs corretas para redirecionamento</h2>
<table>
  <tr>
    <th>Expressão</th>
    <th>Resultado</th>
    <th>Uso para next</th>
  </tr>
  <tr>
    <td>request.path</td>
    <td>/notifications/render</td>
    <td>Recomendado ✅</td>
  </tr>
  <tr>
    <td>request.url</td>
    <td>http://localhost:5000/...</td>
    <td>Não usar ❌</td>
  </tr>
  <tr>
    <td>request.base_url</td>
    <td>http://localhost:5000/...</td>
    <td>Não usar ❌</td>
  </tr>
  <tr>
    <td>request.full_path</td>
    <td>/notifications/render?x=1</td>
    <td>Cuidado ⚠️</td>
  </tr>
</table>

<hr>

<h2 id="checklist">6. Checklist de boas práticas</h2>
<ul>
  <li><code>next</code> via hidden field ou sessão, nunca URL externa</li>
  <li>AJAX não altera <code>next</code></li>
  <li><code>unauthorized_handler</code> centraliza redirecionamento</li>
  <li>Valide <code>next</code> com <code>is_safe_url</code></li>
  <li>Separar fluxo humano vs requisição de background</li>
</ul>

<hr>

<h2>7. Dicas extras</h2>
<ul>
  <li>Rotas AJAX devem responder apenas dados, nunca HTML completo</li>
  <li>Use status code <code>401</code> para não autenticado</li>
  <li>Nunca use <code>request.url</code> como <code>next</code></li>
  <li>Evite sobrescrever <code>next</code> em polling</li>
</ul>

<hr>

<h2>8. Referências</h2>
<ul>
  <li>Flask-Login: <a href="https://flask-login.readthedocs.io/en/latest/#customizing-the-login-process" target="_blank">Documentação Oficial</a></li>
  <li>Flask Sessions: <a href="https://flask.palletsprojects.com/en/latest/quickstart/#sessions" target="_blank">Documentação Oficial</a></li>
  <li>OWASP Open Redirect: <a href="https://owasp.org/www-community/attacks/Open_redirect" target="_blank">Guia OWASP</a></li>
  <li>MDN — HTTP Status 401: <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401" target="_blank">MDN</a></li>
</ul>

</main>

<footer>
  &copy; 2025 Tutorial Flask Seguro | Desenvolvido por você
</footer>

</body>
</html>
