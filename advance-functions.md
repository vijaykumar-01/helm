### **Advanced Functions in Helm with Examples**

Helm provides several advanced template functions that enhance its capabilities for dynamic chart generation, managing complex logic, and interacting with Kubernetes resources.

---

### **1. `lookup`**
Queries Kubernetes API for resources during the rendering process.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: existing-config
data:
  status: |
    {{- $config := lookup "v1" "ConfigMap" "default" "existing-configmap" }}
    {{- if $config }}
    ConfigMap exists with name: {{ $config.metadata.name }}
    {{- else }}
    ConfigMap does not exist.
    {{- end }}
```

- **Use Case:** Check if a resource exists before applying changes.

---

### **2. `tpl`**
Renders a value as a template. Useful for rendering dynamic content from `values.yaml`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ tpl .Values.nameTemplate . }}
spec:
  replicas: 2
```

- **`values.yaml`:**
  ```yaml
  nameTemplate: "{{ .Release.Name }}-app"
  ```
- **Rendered Output:**
  ```yaml
  metadata:
    name: my-release-app
  ```

---

### **3. `required`**
Ensures critical values are provided. If a value is missing, it will throw an error.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ required "ConfigMap name is required" .Values.configName }}
data:
  key: value
```

- **Error Example:**  
  If `configName` is not set in `values.yaml`, the rendering process fails with the message: `ConfigMap name is required`.

---

### **4. `include`**
Includes and renders another template.

```yaml
{{- define "mychart.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end -}}

metadata:
  name: {{ include "mychart.fullname" . }}
```

- **Output:**
  ```yaml
  metadata:
    name: my-release-mychart
  ```

---

### **5. `toYaml` with Custom Formatting**
Converts a complex map or array to YAML format, preserving structure.

```yaml
env:
{{ toYaml .Values.env | nindent 2 }}
```

- **`values.yaml`:**
  ```yaml
  env:
    - name: ENV
      value: production
    - name: DEBUG
      value: "false"
  ```
- **Output:**
  ```yaml
  env:
    - name: ENV
      value: production
    - name: DEBUG
      value: "false"
  ```

---

### **6. `b64enc` and `b64dec`**
Encodes or decodes Base64 strings, often used for managing secrets.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  password: {{ .Values.password | b64enc }}
```

- **`values.yaml`:**
  ```yaml
  password: mypassword
  ```
- **Rendered Output:**
  ```yaml
  data:
    password: bXlwYXNzd29yZA==
  ```

---

### **7. `hasKey` and `pluck`**
Checks for the existence of a key or retrieves a specific key's value.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  {{- if hasKey .Values "extraConfig" }}
  extraConfig: {{ .Values.extraConfig }}
  {{- end }}
```

- **Use Case:** Dynamically add optional fields to a manifest.

---

### **8. `dict`**
Creates dictionaries dynamically in templates.

```yaml
{{- $labels := dict "app" "myapp" "version" "v1" }}
metadata:
  labels:
{{- range $key, $value := $labels }}
    {{ $key }}: {{ $value }}
{{- end }}
```

- **Output:**
  ```yaml
  metadata:
    labels:
      app: myapp
      version: v1
  ```

---

### **9. `merge`**
Merges two dictionaries.

```yaml
{{- $defaultLabels := dict "app" "myapp" }}
{{- $customLabels := .Values.customLabels | default dict }}
{{- $finalLabels := merge $defaultLabels $customLabels }}
metadata:
  labels:
    {{- range $key, $value := $finalLabels }}
    {{ $key }}: {{ $value }}
    {{- end }}
```

- **Use Case:** Combine default and user-defined labels.

---

### **10. `printf`**
Formats a string with dynamic placeholders.

```yaml
metadata:
  name: {{ printf "%s-%s" .Release.Name .Chart.Name }}
```

- **Output:**  
  If the release name is `my-release` and chart name is `mychart`, the result is:
  ```yaml
  metadata:
    name: my-release-mychart
  ```

---

### **11. `range`**
Loops over arrays or maps.

```yaml
containers:
{{- range .Values.containers }}
  - name: {{ .name }}
    image: {{ .image }}
{{- end }}
```

- **`values.yaml`:**
  ```yaml
  containers:
    - name: app
      image: nginx:1.17
    - name: sidecar
      image: busybox
  ```
- **Output:**
  ```yaml
  containers:
    - name: app
      image: nginx:1.17
    - name: sidecar
      image: busybox
  ```

---

### **12. `semverCompare`**
Compares semantic version strings.

```yaml
{{ if semverCompare ">= 1.20.0" .Capabilities.KubeVersion.GitVersion }}
apiVersion: networking.k8s.io/v1
{{ else }}
apiVersion: networking.k8s.io/v1beta1
{{ end }}
```

- **Use Case:** Adjust manifests based on Kubernetes version.

---

### **13. `trim`, `trimSuffix`, `trimPrefix`**
Removes unwanted characters.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name | trimSuffix "-" }}
```

- **Output:** Removes the trailing `-` from the release name.

---

### **14. `regexMatch` and `regexReplaceAll`**
Performs regex operations.

```yaml
{{ if regexMatch "app.*" .Values.appName }}
App name matches the pattern.
{{ end }}

name: {{ regexReplaceAll "app" "service" .Values.appName }}
```

- **Use Case:** Validate or transform values based on patterns.

---

### **15. `include` with `required`**
Ensure included templates are rendered properly.

```yaml
metadata:
  name: {{ include "mychart.fullname" . | required "fullname is required" }}
```

---

These advanced functions allow for flexible and robust Helm charts. Let me know if you'd like specific examples or have additional requirements! 😊
