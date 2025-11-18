import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class Main {
    
    private static final Map<Character, String> KEY3_MAP = new HashMap<>();
    private static final Map<Character, String> KEY2_MAP = new HashMap<>();
    private static final Map<String, String> LETTER_TO_HEX = new HashMap<>();
    
    static {
        KEY3_MAP.put('_', "BH");
        KEY3_MAP.put('X', "LH");
        KEY3_MAP.put('5', "YN");
        KEY3_MAP.put('J', "PH");
        KEY3_MAP.put('h', "PD");
        KEY3_MAP.put('2', "HD");
        KEY3_MAP.put('9', "PP");
        KEY3_MAP.put('b', "PN");
        KEY3_MAP.put('L', "YL");
        KEY3_MAP.put('4', "PL");
        KEY3_MAP.put('I', "PE");
        
        KEY2_MAP.put('_', "ZQ");
        KEY2_MAP.put('X', "MM");
        KEY2_MAP.put('5', "LI");
        KEY2_MAP.put('J', "EE");
        KEY2_MAP.put('h', "FF");
        KEY2_MAP.put('2', "AK");
        KEY2_MAP.put('9', "VR");
        KEY2_MAP.put('b', "QS");
        KEY2_MAP.put('L', "PM");
        KEY2_MAP.put('4', "CQ");
        KEY2_MAP.put('I', "XR");
        
        LETTER_TO_HEX.put("BH", "E4");
        LETTER_TO_HEX.put("PE", "C1");
        LETTER_TO_HEX.put("LI", "85");
        LETTER_TO_HEX.put("PL", "C8");
        LETTER_TO_HEX.put("PD", "C0");
        LETTER_TO_HEX.put("PM", "C9");
        LETTER_TO_HEX.put("PP", "CC");
        LETTER_TO_HEX.put("PN", "CA");
        LETTER_TO_HEX.put("AK", "D7");
        LETTER_TO_HEX.put("PH", "C4");
        LETTER_TO_HEX.put("LH", "84");
        LETTER_TO_HEX.put("RF", "7E");
        LETTER_TO_HEX.put("BR", "E9");
        LETTER_TO_HEX.put("ND", "B7");
        LETTER_TO_HEX.put("JA", "6F");
        LETTER_TO_HEX.put("CL", "A3");
        LETTER_TO_HEX.put("LE", "A7");
        LETTER_TO_HEX.put("DG", "C3");
        LETTER_TO_HEX.put("OB", "89");
        LETTER_TO_HEX.put("GP", "76");
    }
    
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        System.out.print("Enter ciphertext: ");
        String ciphertext = scanner.nextLine().trim();
        
        int[] key = null;
        while (key == null) {
            System.out.print("Enter key (" + ciphertext.length() + " digits, only 2 or 3): ");
            String keyInput = scanner.nextLine().trim();
            
            String[] keyParts = keyInput.split("\\s+");
            
            if (keyParts.length != ciphertext.length()) {
                System.out.println("Error! Key must have " + ciphertext.length() + " digits.");
                continue;
            }
            
            key = new int[keyParts.length];
            boolean valid = true;
            
            for (int i = 0; i < keyParts.length; i++) {
                try {
                    int digit = Integer.parseInt(keyParts[i]);
                    if (digit == 2 || digit == 3) {
                        key[i] = digit;
                    } else {
                        System.out.println("Error! Use only 2 or 3.");
                        valid = false;
                        key = null;
                        break;
                    }
                } catch (NumberFormatException e) {
                    System.out.println("Error! Invalid input.");
                    valid = false;
                    key = null;
                    break;
                }
            }
        }
        
        System.out.println("\n=== DECRYPTION PROCESS ===\n");
        
        String plaintext = decrypt(ciphertext, key);
        
        System.out.println("\n=== RESULT ===");
        System.out.println("Plaintext: " + plaintext);
        
        scanner.close();
    }
    
    private static String decrypt(String ciphertext, int[] key) {
        System.out.println("STAGE 1: Symbol to Letter Pair");
        String[] letterPairs = new String[ciphertext.length()];
        boolean[] skipIndexes = new boolean[ciphertext.length()];
        
        for (int i = 0; i < ciphertext.length(); i++) {
            char symbol = ciphertext.charAt(i);
            int keyValue = key[i];
            
            String letterPair = (keyValue == 3) ? KEY3_MAP.get(symbol) : KEY2_MAP.get(symbol);
            letterPairs[i] = letterPair;
            
            if (LETTER_TO_HEX.get(letterPair) == null) {
                skipIndexes[i] = true;
                System.out.printf("  %c (key=%d) -> %s (not in table, will be '_')%n", symbol, keyValue, letterPair);
            } else {
                System.out.printf("  %c (key=%d) -> %s%n", symbol, keyValue, letterPair);
            }
        }
        System.out.println();
        
        System.out.println("STAGE 2: Letter Pair to Hex");
        String[] hexValues = new String[letterPairs.length];
        for (int i = 0; i < letterPairs.length; i++) {
            if (skipIndexes[i]) {
                hexValues[i] = null;
                System.out.printf("  %s -> skipped%n", letterPairs[i]);
            } else {
                hexValues[i] = LETTER_TO_HEX.get(letterPairs[i]);
                System.out.printf("  %s -> %s%n", letterPairs[i], hexValues[i]);
            }
        }
        System.out.println();
        
        System.out.println("STAGE 3: XOR with A5");
        String[] transformedHex = new String[hexValues.length];
        for (int i = 0; i < hexValues.length; i++) {
            if (skipIndexes[i]) {
                transformedHex[i] = null;
                System.out.printf("  skipped%n");
            } else {
                int value = Integer.parseInt(hexValues[i], 16);
                int result = value ^ 0xA5;
                transformedHex[i] = String.format("%02X", result);
                System.out.printf("  %s XOR A5 = %s%n", hexValues[i], transformedHex[i]);
            }
        }
        System.out.println();
        
        System.out.println("STAGE 4: Hex to ASCII");
        StringBuilder plaintext = new StringBuilder();
        for (int i = 0; i < transformedHex.length; i++) {
            if (skipIndexes[i]) {
                plaintext.append('_');
                System.out.printf("  '_' (replaced)%n");
            } else {
                int value = Integer.parseInt(transformedHex[i], 16);
                char ch = (char) value;
                plaintext.append(ch);
                System.out.printf("  %s -> '%c'%n", transformedHex[i], ch);
            }
        }
        
        return plaintext.toString();
    }
}
