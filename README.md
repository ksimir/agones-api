# How to call Agones API
This repo explains how to connect to the Agones API using REST

In Agones, many setup taks can be done appying YAML config using kubectl and this works great for most testint scenarios. However, in production, you might want ot automate DGS (Dedicated Game Server) allocation, listing, etc calling directly the Agones API.

I tried to gather the step by step below to make it work:

## Create a ServiceAccount, Role and RoleBinding
To simplify this process, just apply the following yaml file
```
kubectl apply -f https://raw.githubusercontent.com/ksimir/agones-api/master/service-account.yaml
```

If you want to grant more or less permissions to the service account, modify the YAML file (in the Role section).

Once applied, the role can be edited using the following command:
```
kubectl edit role agones-api-explorer
```

## Get the Bearer Token, Certificate and API Server URL
Get the token and certificate from the ServiceAccount’s token secret for use in your API requests.  
Start by setting the SERVICE_ACCOUNT variable.
```
SERVICE_ACCOUNT=agones-api-explorer
```

Get the ServiceAccount's token Secret's name
```
SECRET=$(kubectl get serviceaccount ${SERVICE_ACCOUNT} -o jsonpath="{.secrets[0].name}")
```

Extract the Bearer token from the Secret and decode
```
TOKEN=$(kubectl get secret ${SECRET} -o jsonpath="{.data.token}"|base64 -D)
```

Extract, decode and write the ca.crt to a temporary location
```
kubectl get secret ${SECRET} -o jsonpath="{.data.ca\.crt}"|base64 -D > /tmp/ca.crt
```
Get the API Server address
```
APISERVER=https://$(kubectl -n default get endpoints kubernetes --no-headers | awk '{ print $2 }')
```

## Explore the Agones API
```
curl $APISERVER/apis/stable.agones.dev/v1alpha1/namespaces/default/fleets  --header "Authorization: Bearer $TOKEN" --cacert /tmp/ca.crt
```
