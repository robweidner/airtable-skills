# Extension Component Patterns

Common UI patterns for Airtable extensions using the @airtable/blocks/ui components.

## Built-in Components

The SDK provides pre-styled components that match Airtable's design:

```jsx
import {
  Box,
  Button,
  Input,
  Select,
  Text,
  Heading,
  Link,
  Icon,
  Loader,
  Dialog,
  ConfirmationDialog,
  Tooltip,
  FormField,
  Label,
  Switch,
  ProgressBar,
  RecordCard,
  TablePicker,
  FieldPicker,
  ViewPicker,
  RecordPicker
} from '@airtable/blocks/ui';
```

## Layout with Box

Box is a flexible container with props for styling:

```jsx
// Flexbox row
<Box display="flex" gap={2} alignItems="center">
  <Input />
  <Button>Submit</Button>
</Box>

// Flexbox column
<Box display="flex" flexDirection="column" gap={2}>
  <Text>Item 1</Text>
  <Text>Item 2</Text>
</Box>

// Padding and margin
<Box padding={3} marginBottom={2}>
  Content
</Box>

// Background and border
<Box
  backgroundColor="lightGray1"
  border="default"
  borderRadius="default"
  padding={3}
>
  Styled box
</Box>
```

## Form Pattern

```jsx
function ContactForm({ onSubmit }) {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  return (
    <Box padding={3}>
      <FormField label="Name">
        <Input
          value={name}
          onChange={e => setName(e.target.value)}
          placeholder="Enter name"
        />
      </FormField>

      <FormField label="Email">
        <Input
          value={email}
          onChange={e => setEmail(e.target.value)}
          type="email"
          placeholder="Enter email"
        />
      </FormField>

      <Box marginTop={3}>
        <Button
          variant="primary"
          onClick={() => onSubmit({ name, email })}
          disabled={!name || !email}
        >
          Save
        </Button>
      </Box>
    </Box>
  );
}
```

## Table/Field Pickers

Let users select tables and fields from the base:

```jsx
function TableSelector() {
  const [table, setTable] = useState(null);
  const [field, setField] = useState(null);

  return (
    <Box>
      <FormField label="Select table">
        <TablePicker
          table={table}
          onChange={newTable => {
            setTable(newTable);
            setField(null); // Reset field when table changes
          }}
        />
      </FormField>

      {table && (
        <FormField label="Select field">
          <FieldPicker
            table={table}
            field={field}
            onChange={setField}
            allowedTypes={['singleLineText', 'email']} // Optional filter
          />
        </FormField>
      )}
    </Box>
  );
}
```

## Record Picker

Let users select records from a table:

```jsx
function RecordSelector({ table }) {
  const [record, setRecord] = useState(null);

  return (
    <FormField label="Select contact">
      <RecordPicker
        table={table}
        record={record}
        onChange={setRecord}
        placeholder="Choose a contact"
      />
    </FormField>
  );
}
```

## Record Card

Display a record with standard styling:

```jsx
function RecordDisplay({ record }) {
  return (
    <RecordCard
      record={record}
      onClick={() => expandRecord(record)}
    />
  );
}
```

## Loading State

```jsx
function DataLoader() {
  const [loading, setLoading] = useState(true);

  if (loading) {
    return (
      <Box
        display="flex"
        justifyContent="center"
        alignItems="center"
        height="200px"
      >
        <Loader />
      </Box>
    );
  }

  return <DataContent />;
}
```

## Confirmation Dialog

```jsx
function DeleteButton({ record, onDelete }) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <Button
        variant="danger"
        onClick={() => setIsOpen(true)}
      >
        Delete
      </Button>

      {isOpen && (
        <ConfirmationDialog
          title="Delete record?"
          body="This action cannot be undone."
          onConfirm={() => {
            onDelete(record);
            setIsOpen(false);
          }}
          onCancel={() => setIsOpen(false)}
        />
      )}
    </>
  );
}
```

## Settings Panel Pattern

```jsx
function SettingsPanel() {
  const globalConfig = useGlobalConfig();

  const [tableId, setTableId] = useState(
    globalConfig.get('tableId')
  );
  const [showCompleted, setShowCompleted] = useState(
    globalConfig.get('showCompleted') ?? true
  );

  const saveSettings = async () => {
    await globalConfig.setAsync('tableId', tableId);
    await globalConfig.setAsync('showCompleted', showCompleted);
  };

  return (
    <Box padding={3}>
      <Heading size="small" marginBottom={3}>
        Settings
      </Heading>

      <FormField label="Data table">
        <TablePicker
          table={base.getTableByIdIfExists(tableId)}
          onChange={table => setTableId(table?.id)}
        />
      </FormField>

      <FormField label="Show completed items">
        <Switch
          value={showCompleted}
          onChange={setShowCompleted}
        />
      </FormField>

      <Box marginTop={3}>
        <Button onClick={saveSettings}>
          Save Settings
        </Button>
      </Box>
    </Box>
  );
}
```

## Data Table Pattern

```jsx
function DataTable({ records, fields }) {
  return (
    <Box border="default" borderRadius="default" overflow="hidden">
      {/* Header */}
      <Box
        display="flex"
        backgroundColor="lightGray1"
        padding={2}
        borderBottom="default"
      >
        {fields.map(field => (
          <Box key={field.id} flex={1} fontWeight="bold">
            {field.name}
          </Box>
        ))}
      </Box>

      {/* Rows */}
      {records.map(record => (
        <Box
          key={record.id}
          display="flex"
          padding={2}
          borderBottom="default"
        >
          {fields.map(field => (
            <Box key={field.id} flex={1}>
              {record.getCellValueAsString(field.name)}
            </Box>
          ))}
        </Box>
      ))}
    </Box>
  );
}
```

## Search/Filter Pattern

```jsx
function SearchableList({ records }) {
  const [search, setSearch] = useState('');

  const filtered = records.filter(record =>
    record.getCellValueAsString('Name')
      .toLowerCase()
      .includes(search.toLowerCase())
  );

  return (
    <Box>
      <Box marginBottom={3}>
        <Input
          value={search}
          onChange={e => setSearch(e.target.value)}
          placeholder="Search by name..."
        />
      </Box>

      <Box>
        {filtered.length === 0 ? (
          <Text textColor="light">No results found</Text>
        ) : (
          filtered.map(record => (
            <Box key={record.id} padding={2} borderBottom="default">
              {record.getCellValueAsString('Name')}
            </Box>
          ))
        )}
      </Box>
    </Box>
  );
}
```

## Tabs Pattern

```jsx
function TabbedInterface() {
  const [activeTab, setActiveTab] = useState('list');

  return (
    <Box>
      {/* Tab buttons */}
      <Box display="flex" borderBottom="default" marginBottom={3}>
        <Button
          variant={activeTab === 'list' ? 'primary' : 'default'}
          onClick={() => setActiveTab('list')}
        >
          List
        </Button>
        <Button
          variant={activeTab === 'settings' ? 'primary' : 'default'}
          onClick={() => setActiveTab('settings')}
        >
          Settings
        </Button>
      </Box>

      {/* Tab content */}
      {activeTab === 'list' && <ListView />}
      {activeTab === 'settings' && <SettingsView />}
    </Box>
  );
}
```

## Error Handling Pattern

```jsx
function SafeComponent() {
  const [error, setError] = useState(null);

  const handleAction = async () => {
    try {
      setError(null);
      await riskyOperation();
    } catch (err) {
      setError(err.message);
    }
  };

  return (
    <Box>
      {error && (
        <Box
          backgroundColor="redLight1"
          padding={2}
          marginBottom={2}
          borderRadius="default"
        >
          <Text textColor="red">Error: {error}</Text>
        </Box>
      )}

      <Button onClick={handleAction}>
        Do Thing
      </Button>
    </Box>
  );
}
```

## TypeScript Support

Extensions support TypeScript:

```tsx
import { Record, Table, Field } from '@airtable/blocks/models';

interface Props {
  record: Record;
  table: Table;
}

function TypedComponent({ record, table }: Props) {
  const name: string | null = record.getCellValue('Name') as string | null;

  return <div>{name}</div>;
}
```

For full type definitions, install:
```bash
npm install --save-dev @types/react
```

The SDK types are included with `@airtable/blocks`.
