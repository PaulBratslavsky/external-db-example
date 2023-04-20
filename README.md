# How To Add An External Database To Strapi Cloud

Did you know Strapi Cloud supports external databases?

This post will cover how to add an external database to your strapi cloud account.

Although this is a supported feature, it is not something we recommend unless you have a specific reason for doing so. Proceed at your own risk.

## Caveats

- It may impact the performance of running SQL connection over the public internet.
- Network latency. Choose a data center closer to your cloud region.
- Security is your responsibility.
- No support from Strapi Cloud SE

We recommend using the default Strapi Cloud database for most users because it is optimized for Strapi.

But, in your case, you need to use an external database, so let's see how we can set one up on our cloud account.

## Prerequisites

1.  Local Strapi project.
2.  The same project deployed to Strapi Cloud.
3.  Credential for an external database.

You can watch the following [video](https://youtu.be/gsJZNV3trJE) on deploying a new Strapi Project to the cloud. But in this example, I already have a project running locally on my computer.

I have a local project running SQLite database, and the same project deployed on Strapi Cloud with the default PostgreSQL database.

But now I want to point my Strapi project to my external production db hosted on digital ocean.

Let's take a look at how to accomplish this. The steps are similar to how you would do them on a regular project.

But, I wanted to make this walkthrough as a supplementary post to our documentation. [Strapi Docs DB Settings](https://docs.strapi.io/dev-docs/configurations/database#environment-variables-in-database-configurations)

**warning**: before proceeding, make sure you have all your data backed up.

## Current Database Settings

We are currently using `sqlite` database locally and default `postgreql` on Strapi Cloud.

`config/server.js`

```javascript
const path = require("path");

module.exports = ({ env }) => {
  const client = env("DATABASE_CLIENT", "sqlite");
  const connections = {
    mysql: {
      connection: {
        connectionString: env("DATABASE_URL"),
        host: env("DATABASE_HOST", "localhost"),
        port: env.int("DATABASE_PORT", 3306),
        database: env("DATABASE_NAME", "strapi"),
        user: env("DATABASE_USERNAME", "strapi"),
        password: env("DATABASE_PASSWORD", "strapi"),
        ssl: env.bool("DATABASE_SSL", false) && {
          key: env("DATABASE_SSL_KEY", undefined),
          cert: env("DATABASE_SSL_CERT", undefined),
          ca: env("DATABASE_SSL_CA", undefined),
          capath: env("DATABASE_SSL_CAPATH", undefined),
          cipher: env("DATABASE_SSL_CIPHER", undefined),
          rejectUnauthorized: env.bool(
            "DATABASE_SSL_REJECT_UNAUTHORIZED",
            true
          ),
        },
      },

      pool: {
        min: env.int("DATABASE_POOL_MIN", 2),
        max: env.int("DATABASE_POOL_MAX", 10),
      },
    },

    mysql2: {
      connection: {
        host: env("DATABASE_HOST", "localhost"),
        port: env.int("DATABASE_PORT", 3306),
        database: env("DATABASE_NAME", "strapi"),
        user: env("DATABASE_USERNAME", "strapi"),
        password: env("DATABASE_PASSWORD", "strapi"),
        ssl: env.bool("DATABASE_SSL", false) && {
          key: env("DATABASE_SSL_KEY", undefined),
          cert: env("DATABASE_SSL_CERT", undefined),
          ca: env("DATABASE_SSL_CA", undefined),
          capath: env("DATABASE_SSL_CAPATH", undefined),
          cipher: env("DATABASE_SSL_CIPHER", undefined),
          rejectUnauthorized: env.bool(
            "DATABASE_SSL_REJECT_UNAUTHORIZED",
            true
          ),
        },
      },

      pool: {
        min: env.int("DATABASE_POOL_MIN", 2),
        max: env.int("DATABASE_POOL_MAX", 10),
      },
    },

    postgres: {
      connection: {
        connectionString: env("DATABASE_URL"),
        host: env("DATABASE_HOST", "localhost"),
        port: env.int("DATABASE_PORT", 5432),
        database: env("DATABASE_NAME", "strapi"),
        user: env("DATABASE_USERNAME", "strapi"),
        password: env("DATABASE_PASSWORD", "strapi"),

        ssl: env.bool("DATABASE_SSL", false) && {
          key: env("DATABASE_SSL_KEY", undefined),
          cert: env("DATABASE_SSL_CERT", undefined),
          ca: env("DATABASE_SSL_CA", undefined),
          capath: env("DATABASE_SSL_CAPATH", undefined),
          cipher: env("DATABASE_SSL_CIPHER", undefined),
          rejectUnauthorized: env.bool(
            "DATABASE_SSL_REJECT_UNAUTHORIZED",
            true
          ),
        },

        schema: env("DATABASE_SCHEMA", "public"),
      },

      pool: {
        min: env.int("DATABASE_POOL_MIN", 2),
        max: env.int("DATABASE_POOL_MAX", 10),
      },
    },

    sqlite: {
      connection: {
        filename: path.join(
          __dirname,
          "..",
          env("DATABASE_FILENAME", "data.db")
        ),
      },
      useNullAsDefault: true,
    },
  };

  return {
    connection: {
      client,
      ...connections[client],
      acquireConnectionTimeout: env.int("DATABASE_CONNECTION_TIMEOUT", 60000),
    },
  };
};
```

The above file is used to configure your server. And most of the work is already done. But first, we must add your database credential on Strapi Cloud to point to our external database and then redeploy our project.

Let's see how this is done.


## Adding External Database Credentials 

Before saving and pushing changes to github let's log into Strapi Cloud and add our environment variables.

```env
# Database

DATABASE_CLIENT=postgres
DATABASE_HOST=your_host
DATABASE_PORT=your_port
DATABASE_NAME=your_db_name
DATABASE_USERNAME=your_username
DATABASE_PASSWORD=your_password
DATABASE_SSL=true
DATABASE_SSL_REJECT_UNAUTHORIZED=false
DATABASE_SHEMA=public
```

You can add your variables under settings -> variables

![2023-04-14_12-09-34](https://user-images.githubusercontent.com/6153188/233463304-53b2def8-7c53-4f05-b68b-70075f529e4b.png)

Once you added all your required variables, click the save button.

![2023-04-14_14-54-20 1](https://user-images.githubusercontent.com/6153188/233463397-a50c47cd-5308-4a83-8d98-16b98383ff5e.png)

If you have any changes in the application, you can push them to git and trigger a redeploy; otherwise, you can trigger a redeploy after saving your environment variables.

Since I don't have any changes in code, I will redeploy my project manually. 

<img width="1301" alt="2023-04-20_13-14-52" src="https://user-images.githubusercontent.com/6153188/233463549-deb2a27e-2123-4927-a5da-c365c2e2e25e.png">

![Screenshot 2023-04-14 at 3 02 53 PM](https://user-images.githubusercontent.com/6153188/233463643-7b033e04-6dab-4422-a2ff-e38f712436bc.png)

Once your application is finished building, you should now be using your external database.

## Reverting Back To Default Database

If you're using Strapi Cloud project and had previously added environment variables related to an external database, reverting back to the default database can be accomplished quickly. 

All you need to do is remove the previously added environment variables from the Strapi Cloud project dashboard, click the save button that I forgot to click, and redeploy.   This will revert to the default database.

![revert-db](https://user-images.githubusercontent.com/6153188/233463716-aebf5273-094d-49c6-bb84-141a2aa15f0b.gif)

Remember that any data you had on the external database will no longer be accessible after this action. 

## Conclusion 

In conclusion, Strapi Cloud does support external databases. However, it's only recommended if you have a specific reason to do so. 

In this post, we've walked through the process of adding an external database to a Strapi Cloud project, along with the necessary configurations and environment variables. 

We also covered how to revert to the default database. While using an external database may be a viable option for some users, weighing the caveats and understanding the potential risks involved, such as performance impact, network latency, and security concerns, is essential.
