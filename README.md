# IdentityServer4 Issue Idp Claim (Change from "local")

This took me a bit to figure out.  I'm posting the solution because I could not find a good example when I was searching.  Turns out that identityserver looks for Idp and amr in the Claims Principle and if it does not find them, it assigns "local" to Idp and "pwd" to amr.  Those claim types need to be added before sign-in... so you have to override the CreateAsync method and add them manually.

If you want to change the idp or amr claim values in IdentityServer4, follow these steps:

1.  Add a Claims Principal Factory

    public class CustomClaimsPrincipalFactory : UserClaimsPrincipalFactory<UserIdentity, UserIdentityRole>
    {
        public CustomClaimsPrincipalFactory(UserManager<UserIdentity> userManager, RoleManager<UserIdentityRole> roleManager,
                                                    IOptions<IdentityOptions> optionsAccessor)
            : base(userManager, roleManager, optionsAccessor)
        {
        }

        public async override Task<ClaimsPrincipal> CreateAsync(UserIdentity user)
        {
            var principal = await base.CreateAsync(user);

            // Add your claims here
            ((ClaimsIdentity)principal.Identity).AddClaims(new[] { new Claim(JwtClaimTypes.IdentityProvider, "yourprovidernamehere"),
                                                                   new Claim(JwtClaimTypes.AuthenticationMethod, "password")
                                                                 });

            return principal;
        }
    }

2.  In Startup.ConfigureServices, add Claims processor after services.AddIdentity:

            // Add Custom Claims processor
            services.AddScoped<IUserClaimsPrincipalFactory<UserIdentity>, CustomClaimsPrincipalFactory>();
            
That's it.  Super simple!            
