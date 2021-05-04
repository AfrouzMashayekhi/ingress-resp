#Ingress Response Body By lua
1. ADD below yaml to ingress-nginx deployment >= 0.44
```yaml
        volumeMounts:
        - mountPath: /etc/nginx/lua/plugins/rb-lua
          name: lua-plugin
          ...
          
      volumes:
      - configMap:
          defaultMode: 511
          items:
          - key: main.lua
            mode: 511
            path: main.lua
          name: rb-lua
          optional: false
        name: lua-plugin
```
2. Add below to `lua-plugin` configmap:

```yaml
apiVersion: v1
data:
  main.lua: |-
    local ngx = ngx

    local _M = {}

    function _M.body_filter()
      local un = ngx.var.server_name
      ngx.var.resp_body = un
      ngx.var.resp_body = string.sub(ngx.arg[1], 1, 1000)
      if un == "api.uat.algo.ir" then
        local resp_body = string.sub(ngx.arg[1], 1, 1000)
        ngx.ctx.buffered = (ngx.ctx.buffered or "") .. resp_body
        if ngx.arg[2] then
           ngx.var.resp_body = ngx.ctx.buffered
        end
      end
    end

    return _M
kind: ConfigMap
metadata:
  name: rb-lua
  namespace: ingress-nginx
```

3. Enable lua module in [`nginx-configuration `](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#plugins) configmap 
```plugins:rb-lua```
`log-format-upstream`
```{"time": "$time_iso8601", "remote_addr": "$proxy_protocol_addr", "x_forward_for": "$proxy_add_x_forwarded_for", "request_id": "$req_id",
"remote_user": "$remote_user", "bytes_sent": $bytes_sent, "request_time": $request_time, "status": $status, "vhost": "$host", "request_proto": "$server_protocol",
"path": "$uri", "request_query": "$args", "request_length": $request_length, "duration": $request_time,"method": "$request_method", "http_referrer": "$http_referer",
"http_user_agent": "$http_user_agent" ,"req_body":"$request_body" ,"resp_body":"$resp_body"}
```

4. For adding a new ingress to response body add below `annotation` to ingress and add the `un` to the lua module in configmap

```yaml
    nginx.ingress.kubernetes.io/server-snippet: |
      lua_need_request_body on;
      set $resp_body '';
```
```shell
      if ( un == "api.uat.algo.ir" or un == "blah blak") then

```




