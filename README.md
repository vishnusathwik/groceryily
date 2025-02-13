# grocerily
this is a website made with JS and can be used to make shopping lists with adding and removing items and keeping track of them.
you just need to clone this repository and ur good to go 


Thank you !!




import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.ArgumentCaptor;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.jdbc.core.JdbcTemplate;

import javax.sql.DataSource;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.Map;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
public class StaticTransactionTypeCodeDescLoaderTest {

    @Mock
    private DataSource mockDataSource;

    @Mock
    private JdbcTemplate mockJdbcTemplate;

    @Mock
    private ResultSet mockResultSet;


    @Test
    public void testConstructor_loadsDataFromDatabase() throws SQLException {
        // Setup
        when(mockDataSource.getConnection()).thenReturn(null); // Return null to avoid needing a real connection
        when(mockJdbcTemplate.query(anyString(), any())).thenAnswer(invocation -> {
            org.springframework.jdbc.core.RowMapper<Object> rowMapper = invocation.getArgument(1);
            rowMapper.mapRow(mockResultSet, 0); // Simulate mapping a row
            return null; // Return null as the query method does not return anything in this case
        });

        when(mockResultSet.getString("SOURCE_VALUE")).thenReturn("SV1");
        when(mockResultSet.getString("SOURCE_SYSTEM_CD")).thenReturn("SS1");
        when(mockResultSet.getString("CASH_IND")).thenReturn("CI1");
        when(mockResultSet.getString("TXN_TYPE_DESC")).thenReturn("Description 1");

        // Execute
        StaticTransactionTypeCodeDescLoader loader = new StaticTransactionTypeCodeDescLoader(mockDataSource);

        // Assert
        String key = loader.generateKey("SV1", "SS1", "CI1");
        assertEquals("Description 1", loader.getValue("SV1", "SS1", "CI1"));
        assertNotNull(loader.transactionTypeCodeDescTable);
        assertEquals(1, loader.transactionTypeCodeDescTable.size());
    }


    @Test
    public void testConstructor_emptyResult() throws SQLException {
        // Setup
        when(mockDataSource.getConnection()).thenReturn(null);
        when(mockJdbcTemplate.query(anyString(), any())).thenReturn(null); // Simulate empty result

        // Execute
        StaticTransactionTypeCodeDescLoader loader = new StaticTransactionTypeCodeDescLoader(mockDataSource);

        // Assert
        assertNotNull(loader.transactionTypeCodeDescTable);
        assertTrue(loader.transactionTypeCodeDescTable.isEmpty());
    }

    @Test
    public void testGenerateKey() {
        StaticTransactionTypeCodeDescLoader loader = new StaticTransactionTypeCodeDescLoader(mockDataSource); // You don't need a real datasource for this test

        String key1 = loader.generateKey("SV1", "SS1", "CI1");
        assertEquals("SV1/SS1/CI1", key1);

        String key2 = loader.generateKey(null, "SS1", "CI1");
        assertEquals("NULL/SS1/CI1", key2);

        String key3 = loader.generateKey("SV1", null, "CI1");
        assertEquals("SV1/NULL/CI1", key3);

        String key4 = loader.generateKey("SV1", "SS1", null);
        assertEquals("SV1/SS1/NULL", key4);

        String key5 = loader.generateKey(null, null, null);
        assertEquals("NULL/NULL/NULL", key5);
    }

    @Test
    public void testGetValue() throws SQLException {
        // Setup (similar to testConstructor_loadsDataFromDatabase)
        when(mockDataSource.getConnection()).thenReturn(null); // Return null to avoid needing a real connection
        when(mockJdbcTemplate.query(anyString(), any())).thenAnswer(invocation -> {
            org.springframework.jdbc.core.RowMapper<Object> rowMapper = invocation.getArgument(1);
            rowMapper.mapRow(mockResultSet, 0); // Simulate mapping a row
            return null; // Return null as the query method does not return anything in this case
        });
        when(mockResultSet.getString("SOURCE_VALUE")).thenReturn("SV1");
        when(mockResultSet.getString("SOURCE_SYSTEM_CD")).thenReturn("SS1");
        when(mockResultSet.getString("CASH_IND")).thenReturn("CI1");
        when(mockResultSet.getString("TXN_TYPE_DESC")).thenReturn("Description 1");
        StaticTransactionTypeCodeDescLoader loader = new StaticTransactionTypeCodeDescLoader(mockDataSource);

        // Execute & Assert
        assertEquals("Description 1", loader.getValue("SV1", "SS1", "CI1"));
        assertNull(loader.getValue("NonExistent", "SS1", "CI1")); // Test for null return
    }
}
