# Setup Crossplane

Setup Crossplane cli and add it to the PATH, this action will run the install bash script and store the binary in cache for the next run.

## Example

```yaml
uses: crossplane-contrib/setup-crossplane-action@main
with:
  version: v2.1.3 # Version of the Crossplane CLI to install.
```
