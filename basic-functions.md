Helm is a package manager for Kubernetes that uses templates to deploy and manage Kubernetes applications. Helm's functions are written in Go’s template language and provide powerful capabilities for transforming and managing configuration data.

Here are **basic Helm functions with examples**:

---

### **1. `default`**
Sets a default value if a specified value is not provided.

```yaml
replicas: {{ .Values.replicaCount | default 3 }}
```

- If `replicaCount` is undefined, this will use `3` as the default value.

---

### **2. `quote` and `squote`**
Wraps values in double or single quotes.

```yaml
key: {{ .Values.secretKey | quote }}
anotherKey: {{ .Values.password | squote }}
```

- Example output:
  ```yaml
  key: "mySecretKey"
  anotherKey: 'myPassword'
  ```

---

### **3. `required`**
Fails the template rendering if a value is missing.

```yaml
image: {{ required "Image name is required" .Values.image }}
```

- If `.Values.image` is not provided, Helm will throw an error: `"Image name is required"`.

---

### **4. `repeat`**
Repeats a string a specified number of times.

```yaml
password: {{ repeat "*" 8 }}
```

- Output: `********`

---

### **5. `lookup`**
Queries Kubernetes resources.

```yaml
apiVersion: {{ (lookup "v1" "ConfigMap" "default" "my-configmap").apiVersion }}
```

- Looks up the `ConfigMap` named `my-configmap` in the `default` namespace and returns its `apiVersion`.

---

### **6. `toYaml` and `toJson`**
Converts a map or array to YAML or JSON format.

```yaml
env:
{{ toYaml .Values.env | indent 2 }}
```

- Example:
  ```yaml
  env:
    - name: ENV_VAR
      value: prod
  ```

---

### **7. `include`**
Includes another template and renders it.

```yaml
{{ include "mychart.helpers.mytemplate" . }}
```

- Use this to modularize and reuse templates.

---

### **8. `upper` and `lower`**
Transforms strings to uppercase or lowercase.

```yaml
upperCaseValue: {{ .Values.env | upper }}
lowerCaseValue: {{ .Values.name | lower }}
```

- Example output:
  ```yaml
  upperCaseValue: PRODUCTION
  lowerCaseValue: appname
  ```

---

### **9. `nindent`**
Adds indentation while maintaining proper YAML structure.

```yaml
metadata:
  annotations:
{{ .Values.annotations | toYaml | nindent 4 }}
```

- Adds 4 spaces of indentation.

---

### **10. `b64enc` and `b64dec`**
Encodes or decodes values in Base64 (commonly used for secrets).

```yaml
data:
  username: {{ .Values.username | b64enc }}
```

- Example output:
  ```yaml
  data:
    username: dXNlcm5hbWU=
  ```

---

### **11. `tpl`**
Renders a string as a template. Useful for dynamic values in `.Values`.

```yaml
image: {{ tpl .Values.image . }}
```

- If `image: "{{ .Release.Name }}-app"` in `values.yaml`, it renders the release name dynamically.

---

### **12. `hasKey` and `pluck`**
Checks if a map has a key or extracts values from maps.

```yaml
hasKey: {{ hasKey .Values "keyName" }}
configValue: {{ pluck "key1" .Values | first }}
```

---

### **Example Chart Using These Functions**
**`deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount | default 2 }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name | lower }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image | quote }}"
        env:
{{ toYaml .Values.env | nindent 8 }}
```

**`values.yaml`**
```yaml
replicaCount: 3
image: myapp:latest
env:
  - name: ENV
    value: production
```

Rendered output:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: "myapp:latest"
        env:
          - name: ENV
            value: production
```

---

These functions are the building blocks for flexible, reusable, and dynamic Helm charts. Let me know if you need more advanced examples! 😊
