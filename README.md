import java.lang.reflect.Field;
import java.sql.*;
import java.util.List;
import java.util.Map;

public class GenericDBService {

    public <T> void insertList(
            List<T> items,
            String tableName,
            Map<String, String> fieldToColumnMap) throws Exception {

        if (items == null || items.isEmpty()) return;

        Class<?> clazz = items.get(0).getClass();

        StringBuilder columns = new StringBuilder();
        StringBuilder placeholders = new StringBuilder();

        for (String column : fieldToColumnMap.values()) {
            columns.append(column).append(",");
            placeholders.append("?,");
        }

        columns.deleteCharAt(columns.length() - 1);
        placeholders.deleteCharAt(placeholders.length() - 1);

        String sql = "INSERT INTO " + tableName +
                     " (" + columns + ") VALUES (" + placeholders + ")";

        try (Connection conn = DBConnection.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {

            for (T item : items) {
                int index = 1;

                for (String fieldName : fieldToColumnMap.keySet()) {
                    Field field = clazz.getDeclaredField(fieldName);
                    field.setAccessible(true);
                    Object value = field.get(item);
                    ps.setObject(index++, value);
                }

                ps.addBatch();
            }

            ps.executeBatch();
        }
    }
}
