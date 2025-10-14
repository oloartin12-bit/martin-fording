
.menu =`
🛡️ MARTIN FORDING
DMF - Menu Commandes Sécurité. (1 à 20)🛡️

1. .help                - Voir ce menu
2. .status              - Vérifier statut du bot
3. .antispam on/off     - Activer/Désactiver l’anti-spam
4. .block [num]         - Bloquer un numéro suspect
5. .unblock [num]       - Débloquer un numéro
6. .ban [num]           - Signaler un numéro à WhatsApp
7. .purge               - Supprimer les nuisibles du groupe
8. .scan [msg]          - Scanner message suspect
9. .filterlinks on/off  - Filtrer liens dangereux
10. .log                - Journal des menaces
11. .autorespond on/off - Réponses automatiques
12. .setreply           - Définir une réponse automatique
13. .delreply           - Supprimer une réponse auto
14. .listreply          - Lister les réponses auto
15. .floodprotect on/off- Anti-flood     16. .whitelist add [num]- Ajouter numéro sûr
17. .whitelist remove   - Retirer de la whitelist
18. .blacklist add      - Ajouter blacklist
19. .blacklist remove   - Retirer blacklist
20. .shutdown           - Éteindre le bot

🔐 Nissi V2 DMF - Sécurité & Contrôle
    `;
    sock.sendMessage(m.key.remoteJid, { text: menu }, { quoted: m });
  }
};
```keytool -genkey -v -keystore monkeystore.jks -alias mon_alias -keyalg RSA -keysize 2048 -validity 10000
android {
  ...
  signingConfigs {
    release {
      storeFile file("../keystores/monkeystore.jks")
      storePassword "MON_STORE_PASSWORD"
      keyAlias "mon_alias"
      keyPassword "MON_KEY_PASSWORD"
    }
  }
  buildTypes {
    release {
      signingConfig signingConfigs.release
      minifyEnabled true   // ou false (R8/ProGuard)
      proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
  }
}
