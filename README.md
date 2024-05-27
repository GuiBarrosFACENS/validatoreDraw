package com.santander.mpa.domain.validation;

import com.santander.mpa.domain.entity.BilledInvoiceYK;

import java.math.BigDecimal;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

public class WhitelistValidation {

    private static final Map<String, Function<BilledInvoiceYK, Object>> fieldAccessors = new HashMap<>();

    static {
        fieldAccessors.put("participantCode", BilledInvoiceYK::getParticipantCode);
        fieldAccessors.put("finalBeneficiaryrDocumentType", BilledInvoiceYK::getFinalBeneficiaryrDocumentType);
        fieldAccessors.put("finalBeneficiaryDocumentNumber", BilledInvoiceYK::getFinalBeneficiaryDocumentNumber);
        fieldAccessors.put("finalBeneficiaryName", BilledInvoiceYK::getFinalBeneficiaryName);
    }

    public static Map<String, String> validateFields(BilledInvoiceYK billedInvoiceYK) {
        Map<String, String> invalidFields = new HashMap<>();

        if (billedInvoiceYK.isPayd()) {
            for (Map.Entry<String, Function<BilledInvoiceYK, Object>> entry : fieldAccessors.entrySet()) {
                String fieldName = entry.getKey();
                Object fieldValue = entry.getValue().apply(billedInvoiceYK);

                if (isNullOrEmpty(fieldValue)) {
                    invalidFields.put(fieldName, String.format("%s cannot be null or empty!", capitalize(fieldName)));
                }
            }
            //TODO RETORNA QUE O BOLETO NÃO É PAGO
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

    private static String capitalize(String str) {
        if (str == null || str.isEmpty()) {
            return str;
        }
        return Character.toUpperCase(str.charAt(0)) + str.substring(1);
    }

}
