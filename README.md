# How to add Swagger UI to a plain flask API using an OpenAPI specification

This is a simple guide on how to add SwaggerUI to your plain flask API project **without using any additional libraries.**

#### Step 1: Download swagger ui github repo

Go to this link to download swagger UI from [GitHub](https://github.com/swagger-api/swagger-ui/archive/master.zip) and extract it. We are basically interested in the [dist](https://github.com/swagger-api/swagger-ui/tree/master/dist) directory in this archive.

#### Step 2: Copy the files from `dist` to your project directory

- In your project directory create 2 directories `templates` and `static`
- Move `index.html` from `dist` to `templates` directory and rename it to `swaggerui.html`
- Inside `static` directory, create 3 more directories, `css`,`img` and `js`
- Move `.js` files from `dist` to `static/js`
- Move `.css` files from `dist` to `static/css`
- Move `.png` files from `dist` to `static/img`

Your project directory should now look like this
```
.
├── static
│   ├── css
│   │   └── swagger-ui.css
│   ├── img
│   │   ├── favicon-16x16.png
│   │   └── favicon-32x32.png
│   └── js
│       ├── swagger-ui-bundle.js
│       ├── swagger-ui-standalone-preset.js
│       └── swagger-ui.js
└── templates
    ├── swaggerui.html
```

#### Step 3: Edit swaggerui.html and replace all static url with jinja2 template tags

```html
7    <link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='css/swagger-ui.css') }}">
8    <link rel="icon" type="image/png" href="{{ url_for('static', filename='img/favicon-32x32.png') }}" sizes="32x32"/>
9    <link rel="icon" type="image/png" href="{{ url_for('static', filename='img/favicon-16x16.png') }}" sizes="16x16"/>
36   <script src="{{ url_for('static', filename='js/swagger-ui-bundle.js') }}"> </script>
37   <script src="{{ url_for('static', filename='js/swagger-ui-standalone-preset.js') }}"> </script>
```

#### Step 4: Write your API spec in OpenAPI format

- Go to [swagger editor](https://editor.swagger.io/) and write your API spec in YAML
Here is a Sample API spec:


```yaml
openapi: 3.0.0
info:
  version: 1.0.0
  title: Hello API
  description: An API to return hello in requested language

paths:
  /api:
    get:
      tags:
        - Hello
      description: Returns hello in specified language
      parameters:
        - in: query
          name: lang
          required: true
          description: language
          schema:
            type: string
            example: es

      responses:
        '200':
          description: hello in the requested language
          content:
            text/plain:
              schema:
                type: string
                example: hola
```

- Go to `File` menu in Swagger editor and click on `Convert and save as JSON`
- Place the downloaded `openapi.json` file in your projects' `static` directory
- Update the reference for the source json file in `swaggerui.html` file to refer to the spec file in static directory
```javascript     
42  url: "{{ url_for('static', filename='openapi.json') }}",
```

#### Step 5: Return `swaggerui.html` with flask render_template in your preferred route

- Create `hello_api.py` in your project directory with the following code

```python

from flask import Flask, request, jsonify, render_template


app = Flask(__name__)

@app.route('/')
def get_root():
    print('sending root')
    return render_template('index.html')

@app.route('/api/docs')
def get_docs():
    print('sending docs')
    return render_template('swaggerui.html')


@app.route('/api')
def get_api():
    hello_dict = {'en': 'Hello', 'es': 'Hola'}
    lang = request.args.get('lang')
    return jsonify(hello_dict[lang])


app.run(use_reloader=True, debug=False)

```

- Run the app with command: `python3 hello_api.py` from your project directory
- Your SwaggerUI documentation shoule be accessible on http://localhost:5000/api/docs
