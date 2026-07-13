FROM qwen2.5-coder:1.5b

TEMPLATE """{{- if .Suffix }}<|fim\_prefix|>{{ .Prompt }}<|fim\_suffix|>{{ .Suffix }}<|fim\_middle|>

{{- else if .Messages }}

{{- if or .System .Tools }}<|im\_start|>system

{{- if .System }}

{{ .System }}

{{- end }}

{{- if .Tools }}



\# Tools



You may call one or more functions to assist with the user query.



You are provided with function signatures within <tools></tools>:

<tools>

{{- range .Tools }}

{"type": "function", "function": {{ .Function }}}

{{- end }}

</tools>



For each function call, return a json object with function name and arguments within <tool\_call></tool\_call> with NO other text. Do not include any backticks or ```json.

<tool\_call>

{"name": <function-name>, "arguments": <args-json-object>}

</tool\_call>

{{- end }}<|im\_end|>

{{ end }}

{{- range $i, $\_ := .Messages }}

{{- $last := eq (len (slice $.Messages $i)) 1 -}}

{{- if eq .Role "user" }}<|im\_start|>user

{{ .Content }}<|im\_end|>

{{ else if eq .Role "assistant" }}<|im\_start|>assistant

{{ if .Content }}{{ .Content }}

{{- else if .ToolCalls }}<tool\_call>

{{ range .ToolCalls }}{"name": "{{ .Function.Name }}", "arguments": {{ .Function.Arguments }}}

{{ end }}</tool\_call>

{{- end }}{{ if not $last }}<|im\_end|>

{{ end }}

{{- else if eq .Role "tool" }}<|im\_start|>user

<tool\_response>

{{ .Content }}

</tool\_response><|im\_end|>

{{ end }}

{{- if and (ne .Role "assistant") $last }}<|im\_start|>assistant

{{ end }}

{{- end }}

{{- else }}

{{- if .System }}<|im\_start|>system

{{ .System }}<|im\_end|>

{{ end }}{{ if .Prompt }}<|im\_start|>user

{{ .Prompt }}<|im\_end|>

{{ end }}<|im\_start|>assistant

{{ end }}{{ .Response }}{{ if .Response }}<|im\_end|>{{ end }}"""

SYSTEM You are Qwen, created by Alibaba Cloud. You are a helpful assistant.

LICENSE """

