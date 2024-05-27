import java.math.BigDecimal;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;
import java.util.function.Function;

public class WhitelistValidation {

    private static final Map<String, Function<BilledInvoiceYK, Object>> fieldAccessors = new HashMap<>();
    private static final Set<String> nullableFields = Set.of(
            "participantCode",
            "finalBeneficiaryrDocumentType",
            "finalBeneficiaryDocumentNumber",
            "finalBeneficiaryName"
    );

    static {
        fieldAccessors.put("participantCode", BilledInvoiceYK::getParticipantCode);
        fieldAccessors.put("finalBeneficiaryrDocumentType", BilledInvoiceYK::getFinalBeneficiaryrDocumentType);
        fieldAccessors.put("finalBeneficiaryDocumentNumber", BilledInvoiceYK::getFinalBeneficiaryDocumentNumber);
        fieldAccessors.put("finalBeneficiaryName", BilledInvoiceYK::getFinalBeneficiaryName);
        // Add other fields here
        fieldAccessors.put("paymentValue", BilledInvoiceYK::getPaymentValue); // Example of non-nullable field
    }

    public static Map<String, String> validateFields(BilledInvoiceYK billedInvoiceYK) {
        Map<String, String> invalidFields = new HashMap<>();

        // Ensure billedInvoiceYK.isPayd() is true
        if (!billedInvoiceYK.isPayd()) {
            invalidFields.put("isPayd", "Billed invoice must be paid.");
        }

        // Validate all fields
        for (Map.Entry<String, Function<BilledInvoiceYK, Object>> entry : fieldAccessors.entrySet()) {
            String fieldName = entry.getKey();
            Object fieldValue = entry.getValue().apply(billedInvoiceYK);

            if (!nullableFields.contains(fieldName) && isNullOrEmpty(fieldValue)) {
                invalidFields.put(fieldName, String.format("%s cannot be null or empty!", fieldName));
            }
        }

        return invalidFields;
    }

    private static boolean isNullOrEmpty(Object value) {
        if (value == null) {
            return true;
        }
        if (value instanceof String) {
            return ((String) value).trim().isEmpty();
        }
        if (value instanceof BigDecimal) {
            return ((BigDecimal) value).compareTo(BigDecimal.ZERO) <= 0;
        }
        return false;
    }
}
