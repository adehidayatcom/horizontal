# Component Patterns
## Paket Lebaran Mumpuni — Pola Implementasi Komponen

Dokumen ini menjelaskan pola-pola umum untuk membuat komponen yang konsisten dan maintainable dengan Modernize + Paket Lebaran.

---

## 1. Pattern: Form dengan Validasi (Formik + Yup)

**Lokasi:** `src/app/components/admin/[feature]/[Feature]Form.tsx`

### Struktur Lengkap

```typescript
'use client';

import { Formik, Form, Field, FormikHelpers } from 'formik';
import * as Yup from 'yup';
import {
  Box,
  Button,
  TextField,
  Alert,
  CircularProgress,
  Stack,
} from '@mui/material';
import { useState } from 'react';
import { useSomeMutation } from '@/hooks/mutations/useSomeMutation';

// 1. Define types
interface FormValues {
  field1: string;
  field2: number;
}

// 2. Validation schema
const validationSchema = Yup.object({
  field1: Yup.string()
    .required('Field 1 wajib diisi')
    .min(3, 'Minimal 3 karakter'),
  field2: Yup.number()
    .required('Field 2 wajib diisi')
    .positive('Harus lebih dari 0'),
});

// 3. Component
interface FormProps {
  initialValues?: Partial<FormValues>;
  onSuccess?: () => void;
}

export function MyForm({ initialValues, onSuccess }: FormProps) {
  const { mutate, isLoading } = useSomeMutation();
  const [serverError, setServerError] = useState('');

  const handleSubmit = async (
    values: FormValues,
    { setSubmitting }: FormikHelpers<FormValues>
  ) => {
    try {
      setServerError('');
      await mutate(values);
      onSuccess?.();
    } catch (err) {
      setServerError(err.message);
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <Formik
      initialValues={initialValues || { field1: '', field2: 0 }}
      validationSchema={validationSchema}
      onSubmit={handleSubmit}
    >
      {({ errors, touched, isSubmitting, isValid }) => (
        <Form>
          {serverError && (
            <Alert severity="error" sx={{ mb: 2 }}>
              {serverError}
            </Alert>
          )}

          <Stack spacing={2}>
            <Field
              as={TextField}
              fullWidth
              name="field1"
              label="Field 1"
              error={touched.field1 && !!errors.field1}
              helperText={touched.field1 && errors.field1}
            />

            <Field
              as={TextField}
              fullWidth
              name="field2"
              label="Field 2"
              type="number"
              error={touched.field2 && !!errors.field2}
              helperText={touched.field2 && errors.field2}
            />

            <Button
              type="submit"
              variant="contained"
              disabled={isSubmitting || !isValid || isLoading}
              endIcon={isSubmitting ? <CircularProgress size={20} /> : null}
            >
              {isSubmitting ? 'Menyimpan...' : 'Simpan'}
            </Button>
          </Stack>
        </Form>
      )}
    </Formik>
  );
}
```

### Tips & Patterns

- ✅ Validation schema di `lib/validasi/schemas.ts`, reusable di multiple forms
- ✅ Error message dari RPC sudah Bahasa Indonesia
- ✅ Disable button saat submit (cegah double submit)
- ✅ Show loading state jelas
- ✅ Server error di Alert, field error di helperText

---

## 2. Pattern: Data Table dengan React Table + MUI

**Lokasi:** `src/app/components/admin/[feature]/[Feature]Table.tsx`

### Struktur Lengkap

```typescript
'use client';

import {
  useReactTable,
  getCoreRowModel,
  flexRender,
  ColumnDef,
  PaginationState,
} from '@tanstack/react-table';
import {
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  TablePagination,
  CircularProgress,
  Box,
  IconButton,
  Menu,
  MenuItem,
} from '@mui/material';
import MoreVertIcon from '@mui/icons-material/MoreVert';
import EditIcon from '@mui/icons-material/Edit';
import DeleteIcon from '@mui/icons-material/Delete';
import { useState } from 'react';
import { useQuery } from '@/hooks/admin/useQuery';

// 1. Define row type
interface Row {
  id: number;
  nama: string;
  status: string;
  createdAt: string;
}

// 2. Define columns
const columns: ColumnDef<Row>[] = [
  {
    accessorKey: 'nama',
    header: 'Nama',
    cell: (info) => info.getValue(),
  },
  {
    accessorKey: 'status',
    header: 'Status',
    cell: (info) => <StatusBadge status={info.getValue()} />,
  },
  {
    accessorKey: 'createdAt',
    header: 'Dibuat',
    cell: (info) => formatDate(info.getValue()),
  },
  {
    id: 'actions',
    header: 'Aksi',
    cell: (info) => <ActionMenu row={info.row.original} />,
  },
];

// 3. Component
export function MyTable() {
  const [pagination, setPagination] = useState<PaginationState>({
    pageIndex: 0,
    pageSize: 10,
  });

  const { data, isLoading, error } = useQuery({
    pageIndex: pagination.pageIndex,
    pageSize: pagination.pageSize,
  });

  const table = useReactTable({
    data: data?.rows || [],
    columns,
    getCoreRowModel: getCoreRowModel(),
    rowCount: data?.rowCount,
    state: { pagination },
    onPaginationChange: setPagination,
    manualPagination: true,
  });

  if (error) {
    return <Box color="error">{error}</Box>;
  }

  return (
    <Box>
      <TableContainer>
        <Table>
          <TableHead>
            {table.getHeaderGroups().map((headerGroup) => (
              <TableRow key={headerGroup.id}>
                {headerGroup.headers.map((header) => (
                  <TableCell key={header.id}>
                    {header.isPlaceholder
                      ? null
                      : flexRender(
                          header.column.columnDef.header,
                          header.getContext()
                        )}
                  </TableCell>
                ))}
              </TableRow>
            ))}
          </TableHead>
          <TableBody>
            {isLoading ? (
              <TableRow>
                <TableCell colSpan={columns.length} align="center">
                  <CircularProgress />
                </TableCell>
              </TableRow>
            ) : table.getRowModel().rows.length === 0 ? (
              <TableRow>
                <TableCell colSpan={columns.length} align="center">
                  Tidak ada data
                </TableCell>
              </TableRow>
            ) : (
              table.getRowModel().rows.map((row) => (
                <TableRow key={row.id}>
                  {row.getVisibleCells().map((cell) => (
                    <TableCell key={cell.id}>
                      {flexRender(
                        cell.column.columnDef.cell,
                        cell.getContext()
                      )}
                    </TableCell>
                  ))}
                </TableRow>
              ))
            )}
          </TableBody>
        </Table>
      </TableContainer>

      <TablePagination
        rowsPerPageOptions={[10, 25, 50]}
        component="div"
        count={data?.rowCount || 0}
        rowsPerPage={pagination.pageSize}
        page={pagination.pageIndex}
        onPageChange={(_, newPage) =>
          setPagination((old) => ({ ...old, pageIndex: newPage }))
        }
        onRowsPerPageChange={(e) =>
          setPagination((old) => ({
            ...old,
            pageSize: parseInt(e.target.value),
          }))
        }
      />
    </Box>
  );
}

// Helper components
function ActionMenu({ row }: { row: Row }) {
  const [anchorEl, setAnchorEl] = useState<null | HTMLElement>(null);

  return (
    <>
      <IconButton onClick={(e) => setAnchorEl(e.currentTarget)}>
        <MoreVertIcon />
      </IconButton>
      <Menu
        anchorEl={anchorEl}
        open={!!anchorEl}
        onClose={() => setAnchorEl(null)}
      >
        <MenuItem onClick={() => { /* edit */ }}>
          <EditIcon /> Edit
        </MenuItem>
        <MenuItem onClick={() => { /* delete */ }}>
          <DeleteIcon /> Hapus
        </MenuItem>
      </Menu>
    </>
  );
}

function StatusBadge({ status }: { status: string }) {
  const colors: Record<string, 'default' | 'primary' | 'success' | 'error'> = {
    AKTIF: 'success',
    PENDING: 'default',
    SELESAI: 'primary',
  };
  return <Chip label={status} color={colors[status]} />;
}
```

### Tips

- ✅ Manual pagination untuk kontrol penuh
- ✅ Server-side pagination untuk dataset besar
- ✅ Loading state di table body
- ✅ Empty state untuk data kosong
- ✅ Action menu (edit, delete) di kolom terakhir

---

## 3. Pattern: Custom Hook untuk Data Fetching (SWR)

**Lokasi:** `src/hooks/admin/use[Feature].ts`

### Struktur Lengkap

```typescript
import useSWR, { SWRConfiguration } from 'swr';
import { createClient } from '@/lib/supabase/client';
import { featureKeys } from '@/lib/query/keys';
import { mapError } from '@/lib/query/errors';
import { Database } from '@/types/database';

type Feature = Database['public']['Tables']['feature']['Row'];

interface FetchOptions {
  periodeId: number;
  pageIndex?: number;
  pageSize?: number;
  filters?: Record<string, any>;
}

// Query hook
export function useFeature(options: FetchOptions, config?: SWRConfiguration) {
  const supabase = createClient();
  const key = featureKeys.list(options);

  const { data, error, isLoading, mutate } = useSWR(
    key,
    async () => {
      const query = supabase
        .from('feature')
        .select('*')
        .eq('periode_id', options.periodeId);

      if (options.filters?.search) {
        query.ilike('nama', `%${options.filters.search}%`);
      }

      const offset = ((options.pageIndex || 0) * (options.pageSize || 10));
      const { data, error, count } = await query
        .range(offset, offset + (options.pageSize || 10) - 1)
        .order('created_at', { ascending: false });

      if (error) throw mapError(error);
      return { rows: data || [], rowCount: count || 0 };
    },
    { ...config, revalidateOnFocus: false }
  );

  return {
    data,
    isLoading,
    error: error ? mapError(error) : null,
    mutate,
  };
}

// Detail hook
export function useFeatureDetail(id: number, config?: SWRConfiguration) {
  const supabase = createClient();
  const key = featureKeys.byId(id);

  const { data, error, isLoading, mutate } = useSWR(
    key,
    async () => {
      const { data, error } = await supabase
        .from('feature')
        .select('*')
        .eq('id', id)
        .single();

      if (error) throw mapError(error);
      return data;
    },
    config
  );

  return { data, isLoading, error: error ? mapError(error) : null, mutate };
}

// Mutation hook
export function useCreateFeature() {
  const supabase = createClient();
  const { mutate: revalidateList } = useSWR(featureKeys.all());

  const createFeature = async (payload: {
    periode_id: number;
    nama: string;
    // ... other fields
  }) => {
    try {
      const { data, error } = await supabase.rpc('create_feature', payload);

      if (error) throw error;

      // Revalidate affected queries
      revalidateList();

      return data;
    } catch (err) {
      throw new Error(mapError(err));
    }
  };

  return { createFeature };
}
```

### Tips

- ✅ Terpisah: query, detail, mutation
- ✅ Query key dari factory function
- ✅ Error already mapped saat throw
- ✅ Revalidate hanya query yang terdampak
- ✅ Optional pagination parameters

---

## 4. Pattern: Shared Component (Reusable)

**Lokasi:** `src/app/components/shared/[Component].tsx`

```typescript
'use client';

import { Box, BoxProps, styled } from '@mui/material';

// Styled component
export const CardWrapper = styled(Box)(({ theme }) => ({
  backgroundColor: theme.palette.background.paper,
  borderRadius: theme.shape.borderRadius,
  padding: theme.spacing(2),
  boxShadow: theme.shadows[1],
}));

// Reusable component
interface StatusBadgeProps {
  status: 'AKTIF' | 'PENDING' | 'SELESAI' | 'BATAL';
  variant?: 'filled' | 'outlined';
}

export function StatusBadge({ status, variant = 'filled' }: StatusBadgeProps) {
  const colors: Record<string, any> = {
    AKTIF: { bg: '#10b981', color: '#fff' },
    PENDING: { bg: '#f59e0b', color: '#fff' },
    SELESAI: { bg: '#3b82f6', color: '#fff' },
    BATAL: { bg: '#ef4444', color: '#fff' },
  };

  return (
    <Box
      sx={{
        display: 'inline-block',
        px: 1.5,
        py: 0.5,
        borderRadius: 1,
        backgroundColor: colors[status].bg,
        color: colors[status].color,
        fontSize: '0.875rem',
        fontWeight: 600,
      }}
    >
      {status}
    </Box>
  );
}
```

---

## 5. Pattern: Custom MUI Component Wrapper

**Lokasi:** `src/app/components/ui-components/CustomButton.tsx`

```typescript
import { Button, ButtonProps } from '@mui/material';
import { ReactNode } from 'react';

interface CustomButtonProps extends ButtonProps {
  loading?: boolean;
  icon?: ReactNode;
}

export function PrimaryButton({
  loading,
  icon,
  children,
  ...props
}: CustomButtonProps) {
  return (
    <Button
      variant="contained"
      color="primary"
      disabled={loading || props.disabled}
      startIcon={loading ? <Spinner size={20} /> : icon}
      {...props}
    >
      {loading ? 'Memproses...' : children}
    </Button>
  );
}
```

---

## 6. Pattern: Modal/Dialog untuk Konfirmasi

```typescript
'use client';

import {
  Dialog,
  DialogTitle,
  DialogContent,
  DialogActions,
  Button,
  Typography,
} from '@mui/material';

interface ConfirmDialogProps {
  open: boolean;
  title: string;
  message: string;
  onConfirm: () => void;
  onCancel: () => void;
  loading?: boolean;
}

export function ConfirmDialog({
  open,
  title,
  message,
  onConfirm,
  onCancel,
  loading,
}: ConfirmDialogProps) {
  return (
    <Dialog open={open} onClose={onCancel}>
      <DialogTitle>{title}</DialogTitle>
      <DialogContent>
        <Typography>{message}</Typography>
      </DialogContent>
      <DialogActions>
        <Button onClick={onCancel} disabled={loading}>
          Batal
        </Button>
        <Button onClick={onConfirm} variant="contained" disabled={loading}>
          Konfirmasi
        </Button>
      </DialogActions>
    </Dialog>
  );
}
```

---

## Checklist Component Implementation

Sebelum commit komponen baru:

- [ ] Lokasi file sesuai (admin/, reseller/, atau shared/)
- [ ] Props type-safe dengan TypeScript
- [ ] Error handling (if error, show message)
- [ ] Loading state (skeleton, spinner, atau disabled)
- [ ] Responsive design (mobile-friendly)
- [ ] Accessibility (labels, alt text, ARIA)
- [ ] Bahasa Indonesia untuk user message
- [ ] Follow MUI + Modernize styling
- [ ] No hardcoded strings (gunakan constants/enums)
- [ ] Tested di browser (render, interact, error cases)

