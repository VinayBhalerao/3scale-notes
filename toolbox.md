
```
1. Import openapi spec

3scale import openapi -d 3scale-user39 locations.json --override-private-base-url=http://location-service.international.svc:8080 --staging-public-base-url=https://location-user39-api-staging.amp.apps.vsp-277d.open.redhat.com:443 -t location-catalog


2. Create Application Plan

3scale application-plan apply 3scale-user39 location-catalog Gold -n "Gold Plan" --default


3. Create Application

3scale application apply 3scale-user39 1234567890abcdef --account=admin+test@user39.com --name="Secure Application" --description="Created from the CLI" --plan=Gold --service=location-catalog

4. Export Staging URL

STAGING_URL=$(3scale proxy-config show 3scale-user39 location-catalog sandbox |jq -r .content.proxy.sandbox_endpoint)

5. Send curl request

curl -D - -H "api-key: 1234567890abcdef" "$STAGING_URL/locations"

6. Promote to Production

3scale proxy-config promote 3scale-user39 location-catalog
```
