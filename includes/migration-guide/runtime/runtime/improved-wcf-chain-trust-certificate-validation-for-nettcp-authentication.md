### Improved WCF chain trust certificate validation for Net.Tcp certificate authentication

#### Details

.NET Framework 4.7.2 improves chain trust certificate validation when using certificate authentication with transport security with WCF. With this improvement, client certificates that are used to authenticate to a server must be configured for client authentication.  Similarly server certificates that are for the authenticating a server must be configured for server authentication. With this change, if the root certificate is disabled, the certificate chain validation fails. The same change was also made to .NET Framework 3.5 and later versions via Windows security roll-up. You can find more information at [KB 4055269](https://support.microsoft.com/help/4055269/security-only-update-for-net-framework-3-5-1-4-5-2-4-6-4-6-1-4-6-2-4-7).This change is on by default and can be turned off by a configuration setting.

#### Suggestion

- Validate if your server and client certification has the required EKU OID. If not, update your certification.
- Validate if your root certificate is invalid. If so, update the root certificate.
- If you can't update the certificate, you can work around the breaking change temporarily with the following configuration setting. However, opting out of the change will leave your system vulnerable to the security issue.

  ```xml
  <appSettings>
    <add key="wcf:useLegacyCertificateUsagePolicy" value="true" />
  </appSettings>
  ```

| Name    | Value   |
| :------ | :------ |
| Scope   | Minor   |
| Version | 4.7.2   |
| Type    | Runtime |

#### Affected APIs

Not detectable via API analysis.

<!--

#### Affected APIs

Not detectable via API analysis.

-->
