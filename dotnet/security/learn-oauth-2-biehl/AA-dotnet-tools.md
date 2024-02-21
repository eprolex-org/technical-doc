# AA. Outils pour `.net`

## `code_verifier` et `code_challenge`

```cs
// première implémentation de ChatGPT
using System;
using System.Security.Cryptography;
using System.Text;

public class PKCEGenerator
{
    public static string GenerateCodeVerifier(int length)
    {
        const string validChars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-._~";
        var random = new Random();
        var verifier = new StringBuilder(length);

        for (int i = 0; i < length; i++)
        {
            verifier.Append(validChars[random.Next(validChars.Length)]);
        }

        return verifier.ToString();
    }

    public static string GenerateCodeChallenge(string codeVerifier)
    {
        using (var sha256 = SHA256.Create())
        {
            byte[] codeVerifierBytes = Encoding.UTF8.GetBytes(codeVerifier);
            byte[] codeChallengeBytes = sha256.ComputeHash(codeVerifierBytes);
            string codeChallenge = Base64UrlEncode(codeChallengeBytes);
            return codeChallenge;
        }
    }

    private static string Base64UrlEncode(byte[] bytes)
    {
        string base64 = Convert.ToBase64String(bytes);
        string base64Url = base64.Replace('+', '-').Replace('/', '_').TrimEnd('=');
        return base64Url;
    }

    public static void Main(string[] args)
    {
        // Générer un code_verifier aléatoire
        string codeVerifier = GenerateCodeVerifier(64);
        Console.WriteLine("Code Verifier: " + codeVerifier);

        // Générer un code_challenge à partir du code_verifier
        string codeChallenge = GenerateCodeChallenge(codeVerifier);
        Console.WriteLine("Code Challenge: " + codeChallenge);
    }
}
```

Deuxième implémentation :

```cs
using System;
using System.Security.Cryptography;
using System.Text;

public class PKCEGenerator
{
    public static string GenerateCodeVerifier()
    {
        using var rng = RandomNumberGenerator.Create();

            // Générer suffisamment d'octets pour couvrir une plage de 43 à 128 caractères Base64
        byte[] verifierBytes = new byte[64]; // 64 octets couvriront jusqu'à 86 caractères Base64 (64 * 8 / 6)
        rng.GetBytes(verifierBytes);
            rng.GetBytes(verifierBytes);
            return Base64UrlEncode(verifierBytes);

    }

    public static string GenerateCodeChallenge(string codeVerifier)
    {
        using var sha256 = SHA256.Create();

            byte[] challengeBytes = sha256.ComputeHash(Encoding.UTF8.GetBytes(codeVerifier));
            return Base64UrlEncode(challengeBytes);
    }

    private static string Base64UrlEncode(byte[] bytes)
    {
        return Convert.ToBase64String(bytes)
            .TrimEnd('=') // Supprimer les signes "=" de la fin
            .Replace('+', '-') // Remplacer les signes "+" par "-"
            .Replace('/', '_'); // Remplacer les signes "/" par "_"
    }
}
```

