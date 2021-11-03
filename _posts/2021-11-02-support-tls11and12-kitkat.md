---
layout: post
title:  "Add TLSSocketFactory class which tries to enable and enforce TLSv1.1 TLSv1.2 <= KitKat"
author: evermind
categories: [ android, TLS, kitkat ]
---
Here we go again. If you use OkHttpClient.Builder and want to ensure that
4.1-4.4.x android devices also support TLSv1.1 TLSv1.2. you can use below diff.

```diff
From 23f87f6df1b8a66902ebf70af0da89f5b5adb909 Mon Sep 17 00:00:00 2001
From: evermind <abc@non-existant.y>
Date: Tue, 2 Nov 2021 07:47:09 +0100
Subject: [PATCH] Add TLSSocketFactory class which tries to enable and enforce
 TLSv1.1 TLSv1.2 <= KitKat

The inner Helper class implements the creation of a OkHttpClient.Builder
with above mentioned support. So easy drop in place replacement:
-        OkHttpClient.Builder builder = new OkHttpClient.Builder()
+        OkHttpClient.Builder builder = TLSSocketFactoryCompat.Helper.createOkHttpClientBuilder()
---
 .../newpipe/util/TLS12SocketFactory.java      | 190 ++++++++++++++++++
 1 file changed, 190 insertions(+)
 create mode 100644 app/src/main/java/your/great/package/somewhere/TLS12SocketFactory.java

diff --git a/app/src/main/java/your/great/package/somewhere/TLS12SocketFactory.java b/app/src/main/java/your/greate/package/somewhere/TLS12SocketFactory.java
new file mode 100644
index 000000000..e216cb18e
--- /dev/null
+++ b/app/src/main/java/your/great/package/somewhere/TLS12SocketFactory.java
@@ -0,0 +1,190 @@
+package your.great.package.somewhere
+
+
+import android.os.Build;
+import android.util.Log;
+
+import java.io.IOException;
+import java.net.InetAddress;
+import java.net.Socket;
+import java.security.KeyManagementException;
+import java.security.KeyStore;
+import java.security.KeyStoreException;
+import java.security.NoSuchAlgorithmException;
+import java.util.ArrayList;
+import java.util.Arrays;
+import java.util.List;
+
+import javax.net.ssl.SSLContext;
+import javax.net.ssl.SSLSocket;
+import javax.net.ssl.SSLSocketFactory;
+import javax.net.ssl.TrustManager;
+import javax.net.ssl.TrustManagerFactory;
+import javax.net.ssl.X509TrustManager;
+
+import okhttp3.CipherSuite;
+import okhttp3.ConnectionSpec;
+import okhttp3.OkHttpClient;
+
+/**
+ * The whole purpose of this class is to enable TLSv1.1 and TLSv1.2 on devices <=KITKAT.
+ */
+public final class TLS12SocketFactory extends SSLSocketFactory {
+
+    private static final String TAG = TLS12SocketFactory.class.getSimpleName();
+
+    private final String mTLSv11 = "TLSv1.1";
+    private final String mTLSv12 = "TLSv1.2";
+    private final String[] mTlsProtocols = {mTLSv11, mTLSv12};
+
+    private final SSLSocketFactory mInternalSSLSocketFactory;
+
+    private TLS12SocketFactory(final SSLSocketFactory delegate) {
+        mInternalSSLSocketFactory = delegate;
+    }
+
+    public static SSLSocketFactory getInstance() {
+        SSLSocketFactory socketFactory = null;
+        try {
+            final SSLContext context = SSLContext.getInstance("TLS");
+            context.init(null, null, null);
+
+            if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.KITKAT) {
+                socketFactory = new TLS12SocketFactory(context.getSocketFactory());
+            } else {
+                socketFactory = context.getSocketFactory();
+            }
+
+        } catch (NoSuchAlgorithmException | KeyManagementException e) {
+            Log.d(TAG, String.valueOf(e));
+        }
+
+        return socketFactory;
+    }
+
+    public static X509TrustManager getTrustManager() {
+        TrustManagerFactory trustManagerFactory = null;
+        TrustManager[] trustManagers = null;
+        try {
+            trustManagerFactory = TrustManagerFactory.getInstance(
+                    TrustManagerFactory.getDefaultAlgorithm());
+            trustManagerFactory.init((KeyStore) null);
+            trustManagers = trustManagerFactory.getTrustManagers();
+            if (trustManagers.length != 1 || !(trustManagers[0] instanceof X509TrustManager)) {
+                throw new IllegalStateException("Unexpected default trust managers:"
+                        + Arrays.toString(trustManagers));
+            }
+        } catch (NoSuchAlgorithmException | KeyStoreException e) {
+            Log.e(TAG, String.valueOf(e));
+            return null;
+        }
+
+        final X509TrustManager trustManager = (X509TrustManager) trustManagers[0];
+
+        return trustManager;
+    }
+
+    @Override
+    public String[] getDefaultCipherSuites() {
+        return mInternalSSLSocketFactory.getDefaultCipherSuites();
+    }
+
+    @Override
+    public String[] getSupportedCipherSuites() {
+        return mInternalSSLSocketFactory.getSupportedCipherSuites();
+    }
+
+    @Override
+    public Socket createSocket(final Socket socket, final String host,
+                               final int port, final boolean autoClose) throws IOException {
+        return enableTLSOnSocket(mInternalSSLSocketFactory.createSocket(socket, host,
+                port, autoClose));
+    }
+
+    @Override
+    public Socket createSocket(final String host, final int port) throws IOException {
+        return enableTLSOnSocket(mInternalSSLSocketFactory.createSocket(host, port));
+    }
+
+    @Override
+    public Socket createSocket(final String host, final int port, final InetAddress localHost,
+                               final int localPort) throws IOException {
+        return enableTLSOnSocket(mInternalSSLSocketFactory.createSocket(host, port,
+                localHost, localPort));
+    }
+
+    @Override
+    public Socket createSocket(final InetAddress host, final int port) throws IOException {
+        return enableTLSOnSocket(mInternalSSLSocketFactory.createSocket(host, port));
+    }
+
+    @Override
+    public Socket createSocket(final InetAddress address, final int port,
+                               final InetAddress localAddress,
+                               final int localPort) throws IOException {
+        return enableTLSOnSocket(mInternalSSLSocketFactory.createSocket(address, port,
+                localAddress, localPort));
+    }
+
+    /* Utility methods */
+    private Socket enableTLSOnSocket(final Socket socket) {
+        // skip the fix if implementation doesn't provide the TLS version
+        if ((socket instanceof SSLSocket) && isTLS12ProtoAvailable((SSLSocket) socket)) {
+            ((SSLSocket) socket).setEnabledProtocols(mTlsProtocols);
+        }
+
+        return socket;
+    }
+
+    private boolean isTLS12ProtoAvailable(final SSLSocket sslSocket) {
+
+        for (final String protocol : sslSocket.getSupportedProtocols()) {
+            if (protocol.equals(mTLSv11) || protocol.equals(mTLSv12)) {
+                return true;
+            }
+        }
+
+        return false;
+    }
+
+    /**
+     * Helper class to create OkHttpClient.Builder object with TLSv1.1/1.2 on <=KitKat.
+     */
+    public static class Helper {
+
+        public static OkHttpClient.Builder createOkHttpClientBuilder() {
+            final OkHttpClient.Builder builder = new OkHttpClient.Builder();
+
+            if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.KITKAT) {
+                final X509TrustManager trustManager = TLS12SocketFactory.getTrustManager();
+                if (null != trustManager) {
+                    builder.sslSocketFactory(TLS12SocketFactory.getInstance(), trustManager);
+                    enableMoreCipherSuites(builder);
+                } else {
+                    Log.w(TAG, "could not enable modern TLS stuff");
+                }
+            }
+
+            return builder;
+        }
+
+        private static OkHttpClient.Builder enableMoreCipherSuites(
+                final OkHttpClient.Builder builder) {
+            // Try to enable all modern CipherSuites (+2 more)
+            // that are supported on the device.
+            // https://github.com/square/okhttp/issues/4053#issuecomment-402579554
+            final List<CipherSuite> cipherSuites =
+                    new ArrayList<>(ConnectionSpec.MODERN_TLS.cipherSuites());
+            cipherSuites.add(CipherSuite.TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA);
+            cipherSuites.add(CipherSuite.TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA);
+            final ConnectionSpec legacyTLS = new ConnectionSpec.Builder(ConnectionSpec.MODERN_TLS)
+                    .cipherSuites(cipherSuites.toArray(new CipherSuite[0]))
+                    .build();
+
+            builder.connectionSpecs(Arrays.asList(legacyTLS, ConnectionSpec.CLEARTEXT));
+
+            return builder;
+        }
+    }
+}
+
-- 
2.33.1
```

